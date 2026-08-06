---
title: "One Request to Stall Them All: DoS and Noisy Neighbours in the vLLM Scheduler"
date: 2026-07-16 12:00:00 +0100
tags: [security, ai, llm-serving]
categories: [Threats and Attacks]
author: [david_cardoner, secai_team]
pin: false
image:
  path: /assets/img/posts/coverimages/scheduler-dos.svg
---

The [first post in this series](/posts/PrefixCacheLeak/) went after confidentiality: the KV-cache leaked one tenant's prompt to another through timing. This one is about availability. We read the vLLM scheduler and see how a well-shaped stream of requests can starve everyone else on the box, and how preemption, the mechanism that keeps vLLM from running out of memory, doubles as a way to waste its compute.

A note on framing up front. The prefix-cache leak in post one is a documented issue with a shipped fix. What follows is different: it is an availability analysis reasoned from the scheduler source, not a named CVE. The mechanisms are real and the code is quoted directly, but treat the attacks as "here is what the design permits," not "here is a published exploit."

## **One budget, everybody shares it**

A classic server handles one request per worker. vLLM uses continuous batching instead: every engine step, the scheduler looks at everyone in flight and everyone waiting and packs as much work as it can into one fixed token budget (`vllm/v1/core/sched/scheduler.py`):

```python
token_budget = self.max_num_scheduled_tokens
```

That budget is a single number for the whole server. Not per-user, not per-tenant, not per-API-key. Every token of every request draws from the same pool each step. The scheduler fills it in two phases:

1. **Running requests first.** Requests already decoding get their next token. A decode is one token, so a lot of them fit.
2. **Waiting requests next**, with whatever is left. New prompts are admitted from the waiting queue and their prefill, the expensive part, is charged against the remainder.

```python
# First, schedule the RUNNING requests.
while req_index < len(self.running) and token_budget > 0:
    ...
# Next, schedule the WAITING requests.
while (self.waiting or self.skipped_waiting) and token_budget > 0:
    ...
```

The ordering is the thing to notice. Whoever holds running slots is served first every step, and new arrivals get the leftovers. Nowhere in the loop is there a notion of one caller having taken more than its share.

## **Attack 1: budget starvation (the noisy neighbour)**

The first attack needs no cleverness. Flood the endpoint with concurrent requests. Once they are decoding they fill the running set, and phase 1 spends the budget every step before a single victim prompt reaches phase 2.

The victim's request sits in the waiting queue, accepted but never scheduled. Its time-to-first-token climbs while the attacker's requests sail through. To the victim the model just got slow; to the operator, throughput looks great. The attacker is a noisy neighbour who worked out that the budget is first-come, first-served (FCFS) with no per-tenant cap.

> This is denial of service by resource monopolisation, not a crash. Nothing errors. The service is "up" on every dashboard while a targeted user gets nothing. Those are the hardest outages to catch.
{: .prompt-warning }

## **Attack 2: preemption as a compute amplifier**

The second one is more interesting, because it turns a safety feature into a weapon.

The KV-cache is finite. When the scheduler cannot allocate blocks for a request that needs them, it does not fail. It preempts a running request to free memory, then retries:

```python
new_blocks = self.kv_cache_manager.allocate_slots(request, num_new_tokens, ...)
if new_blocks is not None:
    break                       # allocated, we're good
# Otherwise: free memory by preempting someone.
if self.policy == SchedulingPolicy.PRIORITY:
    preempted_req = max(self.running, key=lambda r: (r.priority, r.arrival_time))
else:                           # default FCFS
    preempted_req = self.running.pop()
self._preempt_request(preempted_req, scheduled_timestamp)
```

Then look at what preemption does to the victim (`_preempt_request`):

```python
def _preempt_request(self, request, timestamp):
    self._free_request_blocks(request)      # throw away its KV-cache
    request.status = RequestStatus.PREEMPTED
    request.num_computed_tokens = 0         # <-- every computed token is discarded
    request.num_preemptions += 1
    self.waiting.prepend_request(request)   # back to the queue, from scratch
```

`num_computed_tokens = 0`. A request that had prefilled an 8,000-token prompt and decoded halfway through its answer goes back to zero, and when it resumes it recomputes from the first token. Preemption does not pause a request, it restarts it.

![Preemption resets a request's computed tokens to zero, forcing a full recompute](/assets/img/posts/scheduler-dos/preemption-reset.svg)
_A request 80% of the way through its work is preempted; `num_computed_tokens` snaps to 0 and all of it is recomputed._

That is the amplifier. Drive the server into memory pressure, with many long-context requests or a burst timed to peak load, and you force a run of preemptions. Each victim discards thousands of tokens of finished compute and redoes it, so a modest amount of attacker traffic multiplies the total GPU work the box performs. The eviction that keeps vLLM from OOM-ing is the same lever that burns its compute.

How practical this is depends on the deployment. You need enough concurrent memory demand to force allocation failures, which is easier on a busy multi-tenant endpoint than on an idle one, and the scheduler will also preempt naturally under honest load. The point is not that preemption is a bug, it is that its cost is unbounded and an attacker can aim it.

> The default preemption target matters. Under FCFS the victim is `self.running.pop()`, the most recently added running request. Under `PRIORITY` it is the worst `(priority, arrival_time)`. An attacker who can set request priority, or who just arrives late, can steer who pays the recompute cost.
{: .prompt-info }

## **FCFS that isn't quite FCFS**

One more thing the source gives away. When a request cannot be scheduled this step, the loop does not stop:

```python
# NOTE(woosuk): Here, by doing `continue` instead of `break`,
# we do not strictly follow the FCFS scheduling policy and
# allow the lower-priority requests to be scheduled.
```

`continue`, not `break`. A request that does not fit is skipped and a later, smaller one can be scheduled ahead of it. Good for throughput, but it means "first in line" is not a guarantee. A stream of tiny requests can keep leapfrogging a victim's larger prompt and stretch its wait out, without ever saturating the whole budget.

## **Mitigations: make fairness explicit**

The scheduler optimises for throughput. Fairness and abuse-resistance are yours to add around it:

| Control | What it buys you |
|---|---|
| Per-tenant rate limiting / quotas (in front of vLLM) | Caps any one caller's share of the shared budget. The most important control. |
| `max_num_seqs` / concurrency caps | Bounds how many running slots one source can hold, blunting Attack 1. |
| `PRIORITY` policy with trusted priorities | Protects real users, but only if clients cannot set their own priority. Never trust a client-supplied priority field. |
| Cap `max_model_len` / context length per tier | Limits how much compute a single preemption can waste (Attack 2). |
| Monitor `num_preemptions` and per-tenant TTFT | Preemption storms and one-tenant latency spikes are the signal for an availability attack that never throws an error. |

I built these edge controls as a small gateway in front of vLLM: it maps each API key to a tenant, applies a per-tenant token-bucket rate limit that returns `429` under flood, and isolates each tenant's prefix cache. It's the "fairness lives at the edge, not in the scheduler" idea made concrete: [`gateway.py`](https://github.com/cardoner1993/gpu-kernel-dev/blob/main/projects/vllm-deployment/gateway.py) (`enforce_rate_limit`).

> vLLM's scheduler is a throughput engine, not a fairness engine, and it will let one caller consume the whole box. In a multi-tenant deployment, availability is enforced at the edge with quotas, concurrency caps and trusted priorities, not inside the scheduler loop. Watch `num_preemptions`: a spike is compute being set on fire.
{: .prompt-tip }

## **Reproduce it yourself**

As with the KV-cache attack, none of this needs a GPU, because the behaviour is in the scheduling logic. The PoC ports a miniature `Scheduler` to pure Python: a fixed shared token budget, a finite pool of KV blocks, a two-phase step, and a preemption path that mirrors the real one. It asserts three things: starvation, the preemption reset, and a per-tenant cap that closes Attack 1.

The core is the reset, ported line-for-line from `_preempt_request`:

```python
def _preempt_request(self, req):
    req.num_computed_tokens = 0      # discard ALL finished work (the amplifier)
    req.num_preemptions += 1
    self.running.remove(req)
    self.waiting.insert(0, req)      # prepend_request: back to the queue

def step(self):
    token_budget = self.max_num_scheduled_tokens
    # PHASE 1: running requests first (decode is cheap) -> attacker hogs this
    # PHASE 2: admit waiting prompts with the LEFTOVER budget -> victim starves
```

Running it prints the two attacks landing, then the fix holding:

```text
[1] budget starvation: after 50 steps victim TTFT still NOT served (0/128 prefilled)  OK
[2] preemption reset: victim had 112 tokens computed, preempted -> 0, must recompute 112  (amplified GPU work)  OK
[3] mitigation: per-tenant running cap -> victim prefilled (128/128) by step 1  OK
```

The full script is [`scheduler_dos_poc.py`](https://github.com/cardoner1993/gpu-kernel-dev/blob/main/exercises/vllm/scheduler_dos_poc.py). Claim 2 is the one worth stepping through: the victim finishes prefilling 112 tokens, a burst of attacker prompts arrives needing KV memory, and in a single step the victim drops back to `0`. Every one of those 112 tokens gets computed again. The model is a simplification, decodes and eviction policy are coarser than the real thing, but the reset it demonstrates is exactly the line quoted above.

## **Why read the source**

"Add rate limiting" is advice you could paste onto any service. Reading `scheduler.py` tells you the specific reason it matters here: a shared token budget served running-first (Attack 1), and a preemption path that zeroes `num_computed_tokens` (Attack 2). That second line is the difference between "preemption slows a request a little" and "preemption lets an attacker run up your GPU bill." You cannot size the blast radius without seeing it.

Next in the series: the sampler, where `temperature`, `top_k` and `top_p` have their own edges, from `temperature == 0` determinism to how sampling parameters shape what an output-based attacker can infer.

---

*Grounded in vLLM v1 source: `vllm/v1/core/sched/scheduler.py` (`schedule`, `_preempt_request`).*

---

All rights reserved
