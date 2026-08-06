---
title: "Prefix Cache = Prompt Leak: A Timing Side-Channel Hiding in Your LLM Server"
date: 2026-07-14 12:00:00 +0100
tags: [security, ai, llm-serving]
categories: [Threats and Attacks]
author: [david_cardoner, secai_team]
pin: false
image:
  path: /assets/img/posts/coverimages/prefix-cache-leak.svg
---

vLLM, like every busy LLM server, caches the KV tensors it computes for a prompt so a later request with the same prefix can skip the work. It is a big win and it is on by default. It is also, on a shared endpoint, a way for one user to detect what another user just sent.

The leak is not in the model. It is in the cache lookup, and it surfaces as a latency difference you can measure. This post follows the vLLM v1 source down to the lines that cause it, shows a working probe, and is honest about why the attack is messier in practice than in a clean demo. It is the first of a short series reading the serving layer, the plumbing around the model, for security-relevant behaviour.

## **Why the cache exists**

An LLM turns each prompt token into a key/value (KV) tensor and keeps it in the KV-cache; every generated token attends back over it. Building those entries for the prompt is the *prefill*, and it is the expensive part of a request.

The optimization is simple. If two requests start with the same tokens, their KV entries for that shared span are identical, so you compute them once and reuse them. A shared system prompt or a common few-shot preamble gets paid for by the first request and is free for the rest. vLLM's design doc is blunt about it:

> Prefix caching kv-cache blocks is a popular optimization [...] we cache the kv-cache blocks of processed requests, and reuse these blocks when a new request comes in with the same prefix as previous requests. Since prefix caching is almost a free lunch and won't change model outputs, it has been widely used by many public endpoints (e.g., OpenAI, Anthropic, etc.).
> Source: `docs/design/prefix_caching.md`

The cache is addressed by content. Each fixed-size block of tokens is hashed (`vllm/v1/core/kv_cache_utils.py`):

```python
def hash_block_tokens(hash_function, parent_block_hash, curr_block_token_ids, extra_keys=None):
    if not parent_block_hash:
        parent_block_hash = NONE_HASH
    curr_block_token_ids_tuple = tuple(curr_block_token_ids)
    return BlockHash(
        hash_function((parent_block_hash, curr_block_token_ids_tuple, extra_keys))
    )
```

The hash covers three things: the parent block's hash, the token ids, and `extra_keys`. No user, tenant, or session id appears anywhere in it. Send the same tokens as someone else and you compute the same hash, land on the same cached block, and skip prefill. That is the feature, and it is also the root of the problem.

## **Timing a cache hit**

A hit and a miss cost different amounts of time. A miss runs prefill for the block; a hit just looks it up (`block_pool.py: get_cached_block`). At the request level that shows up as time-to-first-token (TTFT): a request whose prefix is already cached starts answering sooner.

So TTFT quietly answers a question you should not be able to ask:

> *"Has anyone recently sent a prompt starting with these exact tokens?"* Fast means yes, slow means no.

From there:

- **Confirmation.** Guess a sensitive prefix (an internal system prompt, a templated customer record, an API-key format) and time it. A fast response means it was already cached, so someone else sent it.
- **Extraction.** The cache is prefix-structured, since a block's hash folds in its parent's, so you can extend a known prefix one block at a time and let TTFT confirm each guess. Low-entropy tails (fixed templates, sequential IDs) are the realistic targets.
- **Cross-tenant reach.** With no isolation the cache is a single pool, so one tenant's prompts warm blocks another tenant can probe.

The vLLM maintainers describe the same threat in the docs:

> This prevents timing-based attacks where an adversary could infer cached content by observing latency differences.
> Source: `docs/design/prefix_caching.md`, "Cache Isolation for Security"

> Nothing here touches the model. No weights change, no prompt is injected. It is an infrastructure side-channel, the LLM-serving version of a CPU cache-timing attack.
{: .prompt-warning }

One honest caveat before going further. In a lab the hit/miss gap is obvious. A live endpoint is noisier: network jitter, other requests batched alongside yours, and cache eviction all blur the signal. A real attacker averages many probes per guess and picks low-entropy targets. Treat the clean numbers later in this post as the mechanism, not a turnkey exploit.

## **The default is shared**

The side-channel only bites because isolation is opt-in. Here is everything that feeds `extra_keys` (`kv_cache_utils.py: generate_block_hash_extra_keys`):

```python
extra_keys = lora_extra_keys + mm_extra_keys + cache_salt_keys + prompt_embeds_keys
```

A plain text request, with no LoRA adapter, no multimodal input, and no `cache_salt`, produces an empty `extra_keys`. The hash reduces to the tokens alone, and `need_extra_keys()` returns `False`. Every tenant then shares one content-addressed pool. The separator exists; it just is not there until you ask for it.

A quieter one sits alongside. Switch to the faster non-cryptographic `xxhash` backend and vLLM warns:

> Use of a hashing algorithm that is not considered cryptographically secure theoretically increases the risk of hash collisions, which can [...] leak private information in multi-tenant environments.
> Source: `docs/design/prefix_caching.md`

And with `PYTHONHASHSEED` unset, the base `NONE_HASH` comes from `os.urandom(32)`. Fine for unpredictability, but a reminder that the scheme leans on the hash staying unguessable and collision-resistant.

## **The fix: one salt, one trust boundary**

vLLM already ships the fix. It is `cache_salt`, a per-trust-group value mixed into the first block's hash so that only requests with the same salt can reuse each other's blocks:

```python
cache_salt_keys = [request.cache_salt] if (start_token_idx == 0 and request.cache_salt) else []
```

The `start_token_idx == 0` guard means the salt only touches the first block, which looks incomplete until you remember the hashes are chained. Block 0's hash feeds block 1's, block 1's feeds block 2's, and so on, so salting the head shifts every hash after it. One salt isolates the whole sequence.

Send it per request:

```json
{
  "model": "your-model",
  "messages": [{"role": "user", "content": "..."}],
  "cache_salt": "tenant-42-secret"
}
```

In practice, don't trust the client to remember it. In a companion repo I put a small multi-tenant gateway in front of vLLM that maps each API key to a tenant and injects the salt server-side, overwriting whatever the caller sent, so a request that simply omits `cache_salt` can't silently rejoin the shared pool: [`gateway.py`](https://github.com/cardoner1993/gpu-kernel-dev/blob/main/projects/vllm-deployment/gateway.py) (`apply_cache_salt`).

Rough guidance:

| Deployment | Recommended posture |
|---|---|
| Single tenant / single trust domain | Prefix caching on, no salt needed |
| Multi-tenant shared endpoint | Per-tenant `cache_salt`, isolate caches per trust group |
| Regulated / high-sensitivity prompts | Salt, and consider disabling cross-request reuse |

> Prefix caching is safe inside a trust boundary and leaky across one. Pick where that boundary is and salt accordingly, keep the `sha256` default on multi-tenant boxes, and pin `PYTHONHASHSEED`.
{: .prompt-tip }

## **Reproduce it yourself**

You do not need a GPU or even a vLLM install to see it, because the attack is in the hashing, not the CUDA. The PoC ports `hash_block_tokens` to pure Python, models a content-addressed pool, and checks all four claims: the cross-tenant collision, the TTFT gap, a full token-by-token extraction, and `cache_salt` shutting it down.

```python
def hash_block_tokens(parent_block_hash, curr_block_token_ids, extra_keys=None):
    # Faithful port of vllm/v1/core/kv_cache_utils.py::hash_block_tokens.
    # The key is the ENTIRE tuple (parent, tokens, extra_keys). No user id anywhere.
    if not parent_block_hash:
        parent_block_hash = NONE_HASH
    return hash_fn((parent_block_hash, tuple(curr_block_token_ids), extra_keys))

def prefill(self, block_hashes):
    # A cache HIT skips prefill; a MISS pays for it. That delta IS the oracle.
    ttft = 0.0
    for h in block_hashes:
        if h in self.cached:          # get_cached_block hit
            ttft += LOOKUP_COST
        else:
            ttft += PREFILL_COST_PER_BLOCK
            self.cached[h] = self._next_block_id
            self._next_block_id += 1
    return ttft
```

Running it prints the leak, then the recovered secret, then the fix:

```text
[1] content-only hash collides across tenants; cache_salt separates them  OK
[2] TTFT oracle: cold=30.1 vs warm=0.2  -> hit is observable  OK
[3] extracted victim secret via timing alone: 'PIN=4826'  OK
[4] with cache_salt: exact-secret probe times the same as cold  -> oracle closed  OK
```

The full self-checking script is [`prefix_cache_leak_poc.py`](https://github.com/cardoner1993/gpu-kernel-dev/blob/main/exercises/vllm/prefix_cache_leak_poc.py). The extraction step is the interesting one: knowing only the template `"PIN="`, it recovers four secret digits from latency alone, because each guessed block that matches something already computed for another request comes back faster. Remember the caveat above, though: the toy has no jitter, so it succeeds every time. On a real server you would be averaging noisy measurements, which is why low-entropy targets matter.

> **Try it.** Bump `BLOCK_SIZE` to 16 (vLLM's real default) and watch how block *alignment* changes what an attacker can probe: secrets that do not start on a block boundary are much harder to isolate. Alignment is a security parameter, not just a performance one.
{: .prompt-tip }

## **Why read the source**

A checklist could have told you to salt the cache. What it cannot tell you is why the default leaks, what exactly escapes, or how a 20 ms swing in TTFT becomes someone's system prompt. That lives in about forty lines of `kv_cache_utils.py`, and reading it is the difference between following a rule and knowing its blast radius.

Next in the series: the scheduler, where a shared token budget and preemption turn into an availability problem one client can trigger.

---

*Grounded in vLLM v1 source: `vllm/v1/core/kv_cache_utils.py`, `vllm/v1/core/block_pool.py`, and `docs/design/prefix_caching.md`.*

---

All rights reserved
