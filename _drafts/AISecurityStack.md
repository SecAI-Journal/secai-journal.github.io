---
title: The AI Security Stack Okta, CrowdStrike, Snyk and Lakera
date: 2026-06-09 12:00:00 +0100
tags: [security, ai]
categories: [Trends and Technologies]
author: [erica_malafronte, david_cardoner, secai_team]
pin: false
image:
  path: /assets/img/posts/coverimages/ai-security-stack.png
---

## **What Does a Complete Security Posture Look Like in the AI Era?**

For most of the last decade, the question "are we secure?" had a familiar shape. You checked who could log in, you checked what was running on your laptops and servers, and you checked whether the software you shipped contained known vulnerabilities. Three layers, three categories of tooling, a reasonably well-understood map of the attack surface.

Then organisations started embedding **Large Language Models (LLMs)** into the core of their products and workflows, and that map stopped being complete. An AI assistant that can read your documents, call internal APIs, and act on user instructions is not just another application to scan, it is a new and fundamentally different attack surface. Inputs are natural language, behaviour is probabilistic, and the boundary between *data* and *instruction* collapses in ways traditional controls were never designed to handle.

The result is a **four-layer security model** for the AI era. Each layer answers a different question, and each has a category-defining vendor that has become shorthand for the problem it solves:

| Layer | Question it answers | Representative vendor |
|-------|--------------------|----------------------|
| **Identity** | Who is allowed in? | **Okta** |
| **Endpoint** | What is running on our machines? | **CrowdStrike** |
| **Code** | What are we shipping in our software? | **Snyk** |
| **AI / LLM** | What is our model saying and doing? | **Lakera** |

No single layer is sufficient on its own. A perfect identity perimeter does not help if an endpoint is already compromised; flawless endpoint protection does not matter if you ship a vulnerable dependency; and none of the first three layers can tell whether an LLM has just been talked into leaking its system prompt. This is **defense in depth**, updated for a world where one of your applications can be socially engineered.

Let's walk through each layer.

---

## **Layer 1 Identity: Okta**

Identity is the modern perimeter. Once an organisation moves to the cloud and SaaS, the network boundary largely dissolves, and the meaningful question becomes not *"is this request coming from inside the building?"* but *"is this actually the person they claim to be, and are they allowed to do this?"*

**Okta** sits in front of applications as an identity provider, handling:

- **Single Sign-On (SSO)** so users authenticate once against a central, well-defended system rather than against dozens of individually weaker logins.
- **Multi-Factor Authentication (MFA)**, which remains one of the highest-leverage controls against credential theft and phishing.
- **Lifecycle and access governance**, ensuring access is granted on joining, adjusted on role change, and revoked on departure, the unglamorous plumbing that prevents orphaned accounts from becoming an attacker's foothold.

In the AI era, identity matters *more*, not less. AI agents increasingly act on behalf of users and need their own scoped, auditable identities. The question "which agent is allowed to call this API, and on whose authority?" is an identity problem before it is anything else. Treating non-human and agent identities with the same rigour as human ones is fast becoming a core requirement rather than an afterthought.

> **Why this layer first?** If you cannot trust *who* is acting, nothing downstream can be trusted either. Identity is the root of the chain.
{: .prompt-info }

---

## **Layer 2 Endpoint: CrowdStrike**

Even with strong identity, the devices people use, laptops, servers, cloud workloads, can be compromised through malware, a malicious download, or a vulnerability exploited before a patch lands.

**CrowdStrike** operates at this **endpoint** layer through its Falcon platform, providing **Endpoint Detection and Response (EDR)**. Rather than relying solely on signatures of known malware, modern EDR watches *behaviour*: a process spawning a suspicious child, a binary attempting to read credentials from memory, lateral movement across the network. When something looks like an attack in progress, it can alert, isolate the machine, and give responders the timeline they need to understand what happened.

The shift this layer represents is from **prevention only** to **prevention plus detection and response**. The realistic assumption is that some attacks will get through; the goal is to detect them quickly and contain the blast radius before a single compromised endpoint becomes a full breach.

For AI specifically, endpoints are also where models increasingly *run*. Local inference, AI-enabled developer tooling, and agent runtimes execute on machines that need the same behavioural monitoring as any other workload, arguably more, given how much access those agents are routinely granted.

---

## **Layer 3 Code: Snyk**

The third layer moves from *operating* software to *building* it. Modern applications are assembled far more than they are written from scratch: a typical service is a thin layer of original code resting on a deep tree of **open-source dependencies**, each of which can carry its own vulnerabilities.

**Snyk** works at this **code** layer, integrating into the developer workflow and CI/CD pipeline to find and fix security problems *before* they ship:

- **SCA (Software Composition Analysis)** flagging known vulnerabilities (CVEs) in your dependencies and their transitive dependencies.
- **SAST (Static Application Security Testing)** analysing your own source for insecure patterns.
- **Container and Infrastructure-as-Code scanning** catching misconfigurations before they reach production.

This is the practical face of **"shift left"**: catching issues at the point they are cheapest to fix, in the editor or the pull request, rather than after they are running in front of customers. We have written before about how poisoned or malicious code can enter the supply chain; this layer is a key part of the defence against exactly those threats.

AI raises the stakes here in two directions. First, a large and growing share of code is now **AI-generated**, and assistants will happily produce plausible code with subtle vulnerabilities or hallucinated, non-existent packages (an opening for *dependency-confusion* attacks). Second, AI models *are* software, shipped as code and dependencies that need the same scrutiny as everything else. Scanning the code that builds your AI is as important as scanning any other service.

---

## **Layer 4 AI / LLM: Lakera**

The first three layers are mature, well-understood, and broadly deployed. The fourth is the one the previous decade's stack simply does not cover, and it is where AI changes the rules.

Once an LLM is embedded in an application, with access to data, tools, and the ability to take actions, it becomes a target in its own right. The core problem is **prompt injection**: because the model processes instructions and data through the same natural-language channel, an attacker can hide instructions inside content the model reads (a document, a web page, an email) and hijack its behaviour. There is no clean syntactic boundary to enforce, the way there is between SQL code and SQL data.

**Lakera** operates at this **AI / LLM** layer, acting as a guardrail between users, untrusted content, and the model. Broadly, it aims to:

- **Detect and block prompt injection and jailbreak attempts** before they reach the model or before the model acts on them.
- **Filter sensitive data** preventing PII, secrets, or proprietary information from leaking out through model outputs.
- **Enforce content and safety policies** on both inputs and outputs in real time.

This maps directly onto the **[OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/){:target="_blank"}**, the same framework we referenced when discussing data exposure in *[The Hidden Cost of Smarter AI](/posts/AI-Shadow/)*. Prompt injection is **LLM01**, and sensitive information disclosure is **LLM02**, precisely the risks a dedicated LLM-security layer exists to address. Traditional tools have no concept of these threats: a firewall sees well-formed traffic, an EDR sees a normal process, a code scanner sees a legitimate API call. The malice lives in the *meaning of the text*, which is exactly what this layer is built to inspect.

> **Why a separate layer?** Prompt injection is not a bug you can patch once and forget, it is a structural property of how LLMs consume input. It needs a runtime control, not a one-time fix.
{: .prompt-warning }

---

## **Putting the Stack Together**

The four layers are not independent products bolted side by side, they are a chain, and an attacker only needs the weakest link:

1. **Okta** decides *who* gets in.
2. **CrowdStrike** watches *what runs* on the machines once they are in.
3. **Snyk** governs *what you ship* into those machines.
4. **Lakera** controls *what your AI says and does* once it is live.

Picture a single realistic attack path. A phished credential (defended by **identity**) lands on a laptop, where malware tries to establish persistence (caught by the **endpoint** layer). The same organisation ships a service with a vulnerable dependency (flagged at the **code** layer) that exposes an AI assistant, which an attacker then attempts to manipulate via a malicious document (blocked at the **AI** layer). Remove any one control and the path opens up. That is what defense in depth means in practice, and it is why "are we secure?" is never a single-vendor question.

The genuinely new insight is the fourth layer. For years, three categories, identity, endpoint, and code, were a reasonable approximation of a complete posture. The moment LLMs gained access to data and the ability to act, a fourth became non-negotiable. An AI application is the first piece of software in your stack that can be *socially engineered*, and none of the older layers were designed to defend against persuasion.

A complete security posture in the AI era therefore covers all four:

> **Identity → Endpoint → Code → AI.** Who gets in, what runs, what ships, and what the model does. Miss any one, and the stack has a hole.
{: .prompt-tip }

The specific vendor names will change, and they are used here as shorthand for categories rather than endorsements. The *layers*, however, are durable. Whatever tools you choose, the questions they answer are the ones every organisation deploying AI now has to answer for itself.

---

*This article maps security categories to representative vendors for illustration; it is not an endorsement of any specific product. Information is accurate as of 9 June 2026.*

---

All rights reserved
