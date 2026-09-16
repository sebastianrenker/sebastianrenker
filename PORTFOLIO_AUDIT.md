# Portfolio Audit — Renker Ecosystem

Date: 2026-09-16 · Auditor pass: Staff-engineer / security / research / recruiter / VC lens.
Companion documents: [`CLAIMS_AUDIT.md`](CLAIMS_AUDIT.md), [`VISION.md`](VISION.md).

## Executive summary

The portfolio is, to a genuinely unusual degree, **already evidence-first**. Most
READMEs carry explicit status badges (Prototype/Phase-0), honest-limits sections,
and correctly scoped security language ("tamper-evident, not immutable"; "validated,
not authenticated"; "free + unlimited + frontier + 24/7 does not exist"). The
flagship, `renker-core`, has a strong threat model and a passing security test suite.

This pass therefore did **not** need a large rewrite. Its concrete contributions:

1. Corrected the one materially **overstated** claim — "Zero-Knowledge" relay in
   `renkervault` and the profile — to "content-blind / metadata-minimizing", with
   explicit metadata-visibility disclosure. (A ZK claim with no ZK-proof code is the
   kind of thing a skeptical cryptographer flags immediately.)
2. Corrected "manipulationssicher" (tamper-proof) → "tamper-evident" in the profile.
3. Added the missing `SECURITY_PROPERTIES.md` evidence matrix to `renker-core`, each
   row anchored to a real test.
4. Committed **real** benchmark numbers (`renker-core/benchmarks/RESULTS.md`) instead
   of leaving performance unquantified — and read them honestly (not "fast").
5. Wrote this portfolio-level claims/audit/vision layer.

## Core technical narrative

Autonomous agents get real permissions (files, tools, APIs). The LLM is probabilistic
and can be talked around (prompt injection). So the **security decision must live
outside the model**: deterministic, capability-scoped, fail-closed, auditable. That is
`renker-core`. Everything else is either a supporting layer (authz core, agent demo),
a cross-cutting quality gate (CUSTOS), infrastructure (Swarm), research (Continuum), a
crypto prototype (RenkerVault), or an application (Rencora).

## Repository map & classification

| Repo | Role | License | Tests | Verified this pass |
|---|---|---|---|---|
| renker-core | **Flagship** — deterministic authorization kernel | Proprietary (open-core) | 17 files | ✅ 132 passed + benchmarks |
| renker-core-authz | Public Apache-2.0 authz core (shipped subset) | Apache-2.0 | 12 files | ✅ 89 passed |
| renker-agent-demo | Demo: authz securing an agent | Apache-2.0 | 1 file | ✅ 7 passed |
| custos | Evidence-driven agent-claim verification (Claude Code plugin) | MIT | 1 file | ✅ 16 passed |
| renker-swarm | Free multi-agent orchestrator (controlled autonomy) | none found | 3 files | ⚠️ 0 core tests |
| continuum | Research prototype (Phase 0), continual-learning architecture | MIT | 9 files | ✅ 32 passed |
| renkervault | Crypto **prototype** — content-blind E2E chat | MIT | 14 files | ✅ 82 passed (client) |
| rencora | Application / showcase | Proprietary | 18 files | ✅ 77 passed |
| renker-whitepaper | Documentation | none found | — | docs only |
| sebastianrenker / .github.io | Profile & landing | — | — | claims corrected |

## Findings by lens

**Security engineer.** `renker-core` threat model is exemplary: assets, actors, trust
boundaries, explicit non-guarantees (forged identity, host compromise, replay-by-design,
audit crash window). The `SECURITY_PROPERTIES.md` added here makes the test-to-property
mapping explicit. Highest residual risk: **no external review** of any component, and
`renkervault`'s Double-Ratchet is a self-composed protocol (disclosed, but unaudited).

**Staff engineer.** Zero runtime deps in the kernel; frozen public API guarded by a
test; property-based tests (hypothesis). Real technical debt: several redundant
top-level reports in `renker-core` (`PERFECTION_AUDIT.md`, `PERFECTION_REPORT.md`,
`PHASE_2_REPORT.md`, `RENKER_PLATFORM_AUDIT.md`) — consolidation candidates (move to
`docs/archive/`), **not done here** to avoid breaking cross-references without care.

**Research.** `continuum` is correctly labelled Phase-0 architecture prototype — it
does **not** claim demonstrated learning. To make a learning claim later it needs a
defined evaluation: stateless vs memory-only vs learning baseline, fixed seeds,
held-out tasks, multiple runs, leakage control. That is the gate before the word
"learning" is used as a result rather than a hypothesis.

**Recruiter (30-second test).** The profile now reads: *AI security / infrastructure —
deterministic authorization + controlled autonomy*, flagship = `renker-core`. Clear.

**VC / due diligence.** Real thesis, real flagship with tests and measurements, honest
labelling. What does **not** exist and is not claimed: users, revenue, adoption,
external audits, production deployments. See `VISION.md`.

## End-to-end demonstration (reproducible, real output)

`renker-agent-demo` is the canonical adversarial demo. Run:

```bash
pip install -e .    # pulls renker-core-authz
python scripts/run_demo.py
```

Verified this pass — actual output (not mocked): legit read/write → **ALLOW** (ran);
an injected `read ../secrets` and `write ../traversal` → **DENY** (outside capability
scope); an injected `send exfil.eml` → **REQUIRE_APPROVAL** → human **declined** (not
sent); a legit `send reply.eml` → **REQUIRE_APPROVAL** → human **approved** (sent). The
tamper-evident audit log records all six decisions and `audit.verify()` reports
`chain intact`. This demonstrates the exact boundary: *prompt injection changed what the
agent requested, not what was allowed* — and, per the honest framing, an
**approval-gated** authorized action still needs a human, i.e. the kernel does not judge
semantic intent on its own.

## Corrections applied (files changed)

- `renkervault/README.md` (5×), `renkervault/SECURITY.md` (1×): Zero-Knowledge → content-blind.
- `sebastianrenker/README.md` (4×): ZK + tamper-proof + PQ wording; topic tags.
- `renker-core/SECURITY_PROPERTIES.md` (new), `renker-core/benchmarks/RESULTS.md` (new).
- `sebastianrenker/CLAIMS_AUDIT.md`, `PORTFOLIO_AUDIT.md`, `VISION.md` (new).

## Remaining risks (brutally honest)

- **Nothing is independently reviewed.** Every security/crypto property is self-tested.
- `renkervault` ships a self-composed E2E protocol — the highest-consequence unaudited surface.
- Benchmarks are single-machine, single-run, no percentiles.
- **435 tests pass across 7 repos** (this pass), but `renker-swarm`'s orchestrator core
  has **0 automated tests** — its bundled sub-tools' test files collect nothing.
- `renker-swarm` and `renker-whitepaper` have no LICENSE file (owner decision — not set here).

## Recommended next steps (priority order)

1. Add automated tests to the `renker-swarm` orchestrator core (only repo with none).
2. External crypto review (or swap to `libsignal`) for `renkervault` before any real use.
3. Consolidate `renker-core` redundant reports into `docs/archive/` and fix links.
4. Define and run the Continuum learning-vs-memory evaluation before making a learning claim.
5. Add LICENSE to `renker-swarm`; decide license posture for `renker-whitepaper`.
6. Pursue one external security review of `renker-core` — the single biggest credibility lever.
