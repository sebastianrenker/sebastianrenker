# Vision — Renker

> LLMs are probabilistic. Security decisions should not be.

## Problem

AI agents are moving from *generating text* to *taking actions* — reading and writing
files, calling tools and APIs, touching infrastructure. Actions require permissions.
Permissions create a security boundary. But the thing deciding what to do is a
probabilistic model that can be talked around (prompt injection, poisoned tool output).

Putting the security decision *inside* the model means the boundary inherits the
model's non-determinism. That is the gap.

## Approach

A small, deterministic **authorization boundary between the agent and privileged
actions**: capability-scoped, fail-closed, explainable, auditable. Injection can change
*what is requested*; it cannot change *what the capability permits*. Around that core:
verification of agent claims (CUSTOS), controlled multi-agent execution (Swarm), and
research into longer-lived agent memory/learning (Continuum).

## Current evidence (only what exists)

- `renker-core`: deterministic authorization kernel — **132 tests passing**, an
  explicit threat model with stated non-guarantees, and **measured** ~0.73 ms/decision
  on one machine. Zero runtime dependencies.
- `renker-core-authz`: a public, Apache-2.0, dependency-free subset of the primitive.
- `renkervault`: a **prototype** content-blind E2E chat using audited crypto primitives
  (incl. a real ML-KEM-768 PQ-hybrid handshake) — **82 client tests passing**;
  self-composed protocol, not externally audited.

## What is research, not product

- `continuum` — Phase-0 architecture prototype. Continual learning is a **hypothesis**
  with a defined evaluation gate, not a demonstrated result.

## What does NOT exist (and is not claimed)

No users, revenue, customers, adoption, funding, partnerships, production deployments,
external security audits, or academic validation. No TAM projection is offered — it
would be invented. This is an engineering-and-research portfolio, not a company.

## Roadmap — evidence-driven, not feature-driven

1. **Correctness** — done for the core (tests green). Extend to the un-run suites.
2. **Adversarial validation** — expand attack tests; fuzz the policy/path inputs.
3. **Performance** — the `Path.resolve()` scope check is the first optimization target.
4. **Integration** — route a real agent's actions through the guard end-to-end.
5. **External review** — independent security review of the kernel; crypto review of the vault.
6. **Production hardening** — only after 1–5, and only where evidence supports it.

The honest one-line status: *a coherent technical program with a real, tested flagship —
early, self-reviewed, and clearly labelled as such.*
