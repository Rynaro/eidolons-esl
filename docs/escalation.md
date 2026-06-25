# Escalation: Advisory → Forced

**Version:** 1.0  
**Date:** 2026-06-25

---

## The Policy in One Sentence

ESL is **opt-in and advisory by default**; a project escalates conformance from
*advisory* to *blocking* when its **project-aggregate** right-sizing numbers cross
mechanical thresholds. The escalation is governance, not contract: it changes
*enforcement strictness*, never the ESL grammar.

---

## Two Modes (the lever)

The conformance checker — `conformance/esl-conformance.sh` (§8) and its Go sibling
`tonberry verify` — already ships both modes. Escalation reuses them; it adds **no
new code path**, only a decision about *which mode a project runs*.

| Mode | Flag | Behavior | Exit on violation |
|---|---|---|---|
| **advisory** (default) | `--mode warn` | findings reported; the build proceeds | `0` |
| **forced** | `--mode block` | a hard (MUST-level) violation stops the build | `3` |

A consumer with no ESL awareness stays conformant to ECL/EIIS unchanged (§1.4).
A consumer that *adopts* ESL runs `--mode warn` until it crosses the threshold
below; then it SHOULD run `--mode block`.

---

## The Mechanical Threshold (project-aggregate)

The flip predicate **REUSES the §4.2 right-sizing signal family** — no new
vocabulary, no LLM "is this a big project?" judgment. The §4.2 gate scores a
single change; escalation scores the *project* by aggregating those same signals.
A project escalates `advisory → forced` when **ANY** of three conditions holds:

| Signal (project-aggregate) | Trigger | What it proxies |
|---|---|---|
| **change_count** | `≥ N` changes accumulated | sustained use — ESL is load-bearing here |
| **repo_loc** | repository LOC `≥ L` | the §4.2 "system-wide blast radius" lever |
| **full_ratio** | `full`-tier share `≥ R` | the project is structurally doing big/trade-off work |

**Seed defaults** (policy knobs, tunable — NOT constants baked into the checker):

```
N = 10        # change_count threshold
L = 50_000    # repo_loc threshold
R = 0.4       # full_ratio threshold
```

The point is that the trigger is an **observable number**, reusing the
right-sizing signals, so the flip is deterministic and auditable — the same
mechanical mandate §4.2 applies to a single change applies here to the project.

`tonberry assess` computes this aggregate (`{signals, thresholds, tripped[],
recommended_mode}`) and is the read-only **assessment** a project (or a future
nexus verb) consults to decide whether to flip.

---

## Where the Flip Is Recorded (NOT in esl-1.0)

ESL **stays opt-in** (§1.4). The enforcement decision is a *project* concern with
its own lifecycle, so it is recorded **nexus-side**, not in the ESL contract:

- **Recorded in:** the nexus **`eidolons.mcp.lock`** under the `tonberry` entry —
  an `enforcement: advisory|block` field plus the thresholds that produced it.
  This mirrors the harness `--strict` precedent, which records per-host
  `strict_modes` in the lock.
- **NOT recorded in:** `spec/esl-1.0.md` (opt-in by §1.4 and must stay so), and
  **NOT** in `change.json` (per-change, the wrong scope).

The reason is clean layering: `esl-1.0.md` is a *contract* (what the states mean);
*how strictly a given project enforces it* is policy, owned by the nexus, the same
way EIIS layout is a contract and the nexus owns which Eidolons a project installs.

---

## What Ships Where

| Capability | Owner | Status |
|---|---|---|
| `verify --mode warn` / `--mode block` (the lever) | tonberry + bash checker | **ships** |
| `assess` — project-aggregate signal computation + recommendation | tonberry | **ships** |
| Manual flip (maintainer sets `enforcement: block` in the lock) | nexus | **ships** |
| **Auto-flip recording** (a verb reads `assess`, rewrites the lock field) | nexus | **DEFERRED** |
| Measured-compliance auto-flip (advisory-adherence telemetry) | nexus | **DEFERRED** — needs adoption data |

tonberry ships the **assessment + the lever**; it does NOT decide *when* a project
becomes strict, and it does NOT write the lock. The auto-flip *recording* is a
nexus follow-up (the ESL roadmap already defers the `eidolons spec`/`esl` verb
pending real adoption signals). A project is strict-enforceable **today** by a
one-line lock edit; the *automatic* detection is the later, measured layer.

---

## Why Not Auto-Flip Now

Anti-scope discipline: tonberry is a thin tool, and building a policy engine
against zero adoption data would be ceremony the project cannot yet justify
("structure must earn its keep"). The mechanical lever is free — it is the bash
checker's existing `--mode`. The *assessment* is a pure read-only aggregate over
signals the checker already understands. Deferring only the **recording**
(lock-rewrite) hop keeps the contract opt-in and the tool thin while leaving the
escalation observable and one edit away.

---

## Provenance

| Claim | Source | Confidence |
|---|---|---|
| Advisory-default, forced-on-threshold model | FORGE `forge-tonberry-decision.md` Decision 3 | H (locked) |
| Threshold reuses §4.2 signals (no new vocabulary) | spec/esl-1.0.md §4.2 + Decision 3(b) | H (spec + decision) |
| Seed defaults N=10 / L=50k / R=0.4, tunable | Decision 3(b) | H (decision) |
| Flip recorded in `eidolons.mcp.lock`, NOT esl-1.0 | Decision 3(b) + esl-1.0.md §1.4 (opt-in P0) | H (decision + contract) |
| Auto-flip recording deferred to nexus | Decision 3(c) + roadmap.md candidate #1 (`eidolons spec` verb) | H (decision + roadmap) |
| `tonberry assess` is the read-only assessment | tonberry v0.2.0 ops | H (direct) |
