# Claims Audit — Renker Portfolio

A skeptical, evidence-first pass over the strong technical claims in this portfolio.
The goal is not to make the work look bigger — it is to make every claim
**scoped to what the code and tests actually support**.

**Evidence labels** (only the strongest *supported* label is used):

- `TESTED` — automated tests assert it, and they were **run and passed in this audit pass**.
- `TESTS PRESENT` — test files exist but were not executed in this pass.
- `IMPLEMENTED` — code implements it; no dedicated test verified here.
- `MEASURED` — a reproducible measurement is committed.
- `EXPERIMENTAL` / `PROTOTYPE` — runs, but not validated for production.
- `NOT INDEPENDENTLY REVIEWED` — no external audit (true for **every** item below).
- `CORRECTED` — wording was changed in this pass because it overstated the evidence.

Verification environment: Python 3.12.10, Node/vitest, Windows/AMD64, 2026-09-16.

## What was run in this pass

| Repo | Command | Result |
|---|---|---|
| renker-core | `pytest -q` | **132 passed** |
| renker-core | `python benchmarks/bench.py` | measured (see `benchmarks/RESULTS.md`) |
| renker-core-authz | `pytest -q` | **89 passed** |
| renkervault (client) | `npm test` (vitest) | **82 passed** (11 files) |
| continuum | `pytest -q` | **32 passed** |
| rencora | `pytest tests/ -q` | **77 passed** |
| CUSTOS | `pytest -q` | **16 passed** |
| renker-agent-demo | `pytest -q` | **7 passed** |

Portfolio total verified this pass: **435 tests passing** across 7 repos.

Not passing / not applicable: **renker-swarm** — the 3 `test_*.py` files live in
bundled sub-tools (leadtool/terminki/vogelsbergbox) and collect **0 tests**; the
orchestrator core has no automated tests. Honest status: **NO CORE TESTS**.

## Claim-by-claim

| Claim | Repo | Evidence | Status | Recommended wording |
|---|---|---|---|---|
| Deterministic, fail-closed authorization decision outside the LLM | renker-core | `test_security_failure_never_allows_outside_scope`, `test_none_action_denied_without_crash` + 130 more, all passing | **TESTED** | keep |
| Path-traversal / prefix-confusion / case resistance | renker-core | `test_attack_path_traversal`, `test_attack_prefix_confusion`, 400-example property test | **TESTED** | keep |
| Audit is **tamper-evident, not immutable** | renker-core, renker-core-authz, renker-agent-demo | hash-chain + anchor; `test_detects_modified_entry`, `test_detects_tail_truncation`; full-deletion only detected while anchor survives | **TESTED**, correctly scoped | keep — already honest |
| Full authorization decision latency | renker-core | `benchmarks/RESULTS.md`: ~0.73 ms/decision, single machine, single run | **MEASURED** (narrow) | cite as single-machine, no percentiles |
| Cryptographic identity / actor authentication | renker-core | interface only; `THREAT_MODEL.md` §6 states it is **not** closed | **IMPLEMENTED (interface), NOT a guarantee** | keep — already disclaimed |
| Post-quantum **primitive** in handshake (ML-KEM-768) | renkervault | real `@noble/post-quantum` `ml_kem768`, hybrid w/ X25519; crypto tests pass | **TESTED (primitive)** | "PQ-hybrid handshake" ✓ — **not** "post-quantum secure protocol" |
| "Zero-Knowledge" relay | renkervault, profile, renkervault/SECURITY.md | **no ZK-proof code**; relay sees metadata (account IDs, device info, timing) | **CORRECTED** | now "content-blind / metadata-minimizing relay; not zero-knowledge" |
| E2E Double-Ratchet composition | renkervault | 82 vitest tests pass; composition of audited primitives, **self-composed, not externally audited** (disclosed in README) | **TESTED (composition), NOT AUDITED** | keep — already honest |
| "manipulationssicher" (tamper-proof) audit in profile | profile | audit is tamper-**evident** only | **CORRECTED** → "tamper-evident, nicht tamper-proof" | done |
| Prompt-injection addressed | renker-core, profile | kernel confines the *request* to the capability; does **not** "solve" prompt injection | **IMPLEMENTED, scoped** | "injection can change what is requested, never what is allowed" ✓ |
| Continuum learning/adaptation | continuum | 32 tests passing; Phase-0 research framing | **TESTED (as prototype) / RESEARCH** | keep research label; learning is hypothesis, see PORTFOLIO_AUDIT |
| CUSTOS "verifies" agent claims | CUSTOS | 16 tests passing; evidence-driven checks; "test passed" ≠ "bug fixed" | **TESTED** | scope as "evidence, not proof of correctness" |
| Renker Swarm "autonomous" multi-agent | renker-swarm | orchestration loop; roles assigned by config, not emergent; **no core tests** | **IMPLEMENTED, UNTESTED** | "controlled autonomy; assigned roles" — avoid "emergent"; add tests |
| Rencora app enforces renker-core-style boundaries | rencora | 77 tests passing incl. `test_filesystem_security`, `test_capabilities`, `test_desktop_sandbox`, `test_audit_rotation` | **TESTED** | keep; it is the applied showcase |
| renker-flint "provably lowers token cost" | renker-flint | `MEASUREMENT.md` defines a provider-reported falsification protocol; `measurements/` is **empty** (Phase 0 not run) | **CONCEPT / NOT YET MEASURED** | keep "Konzeptphase"; numbers stay `inferred` until a provider-reported baseline exists — no savings claim as fact yet |

## Corrections applied in this pass

1. **renkervault** README/SECURITY.md: removed "Zero-Knowledge" as a property; now
   "content-blind / metadata-minimizing relay", with explicit metadata-visibility disclosure.
2. **profile** README: "Zero-Knowledge-Kommunikation" → "inhaltsblinde E2E-Kommunikation";
   "manipulationssicheren Audit-Log" → "manipulations­erkennenden (tamper-evident)";
   "post-quantum-verschlüsselt" → "Post-Quantum-Hybrid-Handshake (X25519 + ML-KEM-768)";
   topic tags `zero-knowledge` → `metadata-minimization`, `post-quantum` → `post-quantum-hybrid`.
3. **renker-core**: added `SECURITY_PROPERTIES.md` (evidence matrix) and committed
   real benchmark numbers in `benchmarks/RESULTS.md`.

## Still open (not done in this pass)

- Run and record test results for renker-core-authz, continuum, rencora, CUSTOS, renker-swarm.
- Continuum: define the learning-vs-memory evaluation (baseline, seeds, held-out) — see PORTFOLIO_AUDIT.
- No item here is **independently reviewed**. External crypto review (renkervault) and
  external security review (renker-core) are the highest-value next steps.
