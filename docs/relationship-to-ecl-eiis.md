# Relationship to ECL and EIIS

**Version:** 1.0  
**Date:** 2026-06-24

---

## The Three Sibling Contracts Composed

ESL (Eidolons Spec Lifecycle) is a peer of **ECL** (Eidolons Communication Layer) and **EIIS** (Eidolons Install Interface Specification). All three are **contract repos** — they define agreements, not implementations. They compose by reference; they do not overlap.

```
EIIS  (Rynaro/eidolons-eiis)  — the install contract.
ECL   (Rynaro/eidolons-ecl)   — the wire-format contract.
ESL   (Rynaro/eidolons-esl)   — the spec-lifecycle contract.
```

---

## Layer Separation

| Layer | What it specifies | Lives in | Audience |
|---|---|---|---|
| **EIIS** | Install: repo layout, `install.sh` flag contract, manifest schema, host wiring | `eidolons-eiis/` | Eidolon authors; nexus CLI |
| **ECL** | Runtime hand-off: envelope structure, closed 10-performative set, SHA-256 verify gate, trace lineage | `eidolons-ecl/` | Host LLM (Claude Code, Cursor); inter-Eidolon dispatch |
| **ESL** | Spec lifecycle: status vocabulary + transitions, right-sizing gate, maker/checker rule, drift-check + living-spec contract | `eidolons-esl/` | Spec authors; change-workflow infrastructure teams |
| **Eidolon repo** | Methodology, per-Eidolon artifact bodies (what SPECTRA, FORGE, Vivi, etc. actually do) | `Rynaro/{ATLAS,SPECTRA,…}` | Users of each Eidolon |
| **Nexus** | Roster, presets, CLI orchestration | `Rynaro/eidolons` | End users; integration point for contracts |
| **Consumer project** | `eidolons.yaml`, `eidolons.lock`, installed Eidolons, `.spectra/changes/` folder tree | User-owned | End user + Eidolons ecosystem |

**EIIS knows nothing about runtime hand-offs. ECL knows nothing about install layouts. ESL knows nothing about artifact schemas — it only references them.** The boundary is intentional: each contract can evolve at its own cadence without forcing changes through all three review audiences.

---

## How the Three Compose

### At install time (EIIS)

1. An Eidolon repo satisfies EIIS v1.4 (layout, `install.sh` flag contract, `install.manifest.json` schema).
2. The manifest MAY include an `ecl_version_emitted` field (optional metadata for the nexus to detect ECL awareness).
3. On install, the Eidolon's files land at `./.eidolons/<name>/` in the consumer project.

### At runtime (ECL + ESL)

1. **ECL envelopes are sidecars.** When an Eidolon emits work (a change proposal, a verification result, an archival decision), it wraps the payload in a JSON envelope with:
   - Closed 10-performative performative (PROPOSE, CRITIQUE, DECIDE, DELEGATE, ACKNOWLEDGE, INFORM, ESCALATE, REFUSE, RESUME, ACCEPT — see ECL spec)
   - `from.eidolon` identity (e.g., `spectra`, `vivi`, `kupo`)
   - SHA-256 integrity over the payload
   - Trace context (call stack)

2. **ESL lifecycle rides ECL.** A change flows through states (`proposed → deliberated → in_progress → verified → archived`). Each state transition is accompanied by an ECL envelope:
   - `proposed` → PROPOSE (SPECTRA emits)
   - `deliberated` → CRITIQUE (FORGE objection) or DECIDE (approval)
   - `in_progress` → DELEGATE (parent → maker)
   - `verified` → INFORM(verify_pass) or ESCALATE(verify_fail)
   - `archived` → ACKNOWLEDGE + INFORM(promotion to Semantic)

3. **Memory ingest via context_delta.** At each ESL transition, the ECL envelope's `context_delta` field carries the change's living spec (Execution layer checkpoint). Promotion to `verified` + `archived` writes to CRYSTALIUM Semantic layer (the durable spec-of-record).

4. **Maker/checker verification.** ESL's core P0 rule — `maker ≠ checker` — is enforced by ECL's SHA-256 verify gate. The consumer's runner (e.g., `eidolons run --verify`) checks that the `verify_pass` envelope's `from.eidolon` ≠ the change's maker identity. Self-verify is a contract violation (exit 3).

---

## Composition Points (the only three where contracts touch)

### 1. Root version stamps

Each contract repo has a root file declaring its version:

- **EIIS:** `EIIS_VERSION` (e.g., `1.4`)
- **ECL:** `ECL_VERSION` (e.g., `1.0`)
- **ESL:** `ESL_VERSION` (e.g., `1.0`)

Mirrors across the sibling repos — no new concept, same pattern applied three times. The nexus reads these to detect which contract versions are in scope.

### 2. Optional metadata field in EIIS manifest

An Eidolon's `install.manifest.json` MAY include an `ecl_version_emitted` field:

```json
{
  "eiis_version": "1.4",
  "ecl_version_emitted": "1.0",
  "name": "spectra",
  "version": "4.9.1"
}
```

This is optional metadata for the nexus to detect and report ECL awareness. Not required for conformance; useful for tooling that wants to filter Eidolons by their runtime capabilities.

### 3. Nexus version-mismatch warnings

On `eidolons sync` or `init`, the nexus MAY read the installed Eidolons' `EIIS_VERSION` and `ECL_VERSION` stamps and report drift if versions are incompatible. This is an integration point only — the contracts themselves are independent.

---

## Versioning & Compatibility

Each contract versions independently via SemVer. A breaking change in one does NOT require a bump in the others.

| EIIS | ECL | ESL | Status |
|---|---|---|---|
| 1.1 | 1.0 | 1.0 | Recommended (current) |
| 1.0 | 1.0 | 1.0 | Supported (v1.0 Eidolons MAY emit ECL) |
| 1.2+ | 1.x | 1.x | Forward-compatible (additive) |

**Reversal conditions (if a sibling changes, this contract may need re-versioning):**
- If SPECTRA changes the schema of `spec.{md,yaml}`, ESL's references (not re-declarations) may need a version bump.
- If ECL adds a new performative or changes the envelope structure, ESL's performative-mapping section may need a version bump.
- If CRYSTALIUM changes the shape of Semantic/Execution layers, ESL's memory-contract references may need a version bump.

When in doubt, re-version the changed contract explicitly. Silent implicit follow is forbidden (FORGE principle: versioning is deliberate, never implicit).

---

## Why Three, Not One

A single combined "Eidolons Specification" would have been smaller. We split them because:

1. **Different audiences, different cadences.**
   - EIIS authors: Eidolon authors and the nexus CLI (install-time, infrequent changes).
   - ECL authors: host LLMs and dispatch tooling (runtime-critical, more frequent changes).
   - ESL authors: spec-workflow infrastructure teams (lifecycle-critical, moderate frequency).

2. **Orthogonal concerns.**
   - EIIS = "where files go and how to install them" (filesystem topology).
   - ECL = "how agents communicate at runtime" (wire format).
   - ESL = "how specs flow through states" (governance + memory binding).

3. **Independent evolution.**
   - A change to ECL's performative set would force every ESL user to re-review the entire ESL contract if they were combined. Separated, only the performative-mapping section (§7 of ESL) needs re-versioning.
   - An EIIS layout change would not affect ESL's state machine. Separated, teams can adopt independently.

---

## Drift Register

If real-world conformance discovers cases where a contract spec is stricter or looser than what live implementors do, those drifts are tracked in each contract's `CHANGELOG.md` under `[Unreleased]` using the convention:

- **D-N format:** `D-1`, `D-2`, etc. (one drift per entry)
- **Entry includes:** the performative/field/rule that drifts, the spec vs. real-world behavior, the confidence level, and any provisional guidance.

ESL v1.0 ships with zero drifts. Entries will accumulate as real Eidolon projects adopt.

---

## Example: A Change Flows Through ESL + ECL + CRYSTALIUM

1. **SPECTRA proposes** a new feature spec → emits `spec.{md,yaml}` + `propose.envelope.json` (ECL PROPOSE performative) under `.spectra/changes/feature-123/`.
2. **Right-sizing gate** runs → scores /12 = 8 → tier = `full` → requires deliberation.
3. **FORGE critiques** the spec → emits `critique.envelope.json` (ECL CRITIQUE) + a revised `propose.envelope.json` (approval, ECL DECIDE).
4. **change.json status** transitions to `deliberated`; the envelope's `context_delta` writes to CRYSTALIUM Execution layer (in-flight working spec).
5. **Vivi implements** in an isolated worktree → emits code + `delegate.envelope.json` (ECL DELEGATE) → status = `in_progress`.
6. **Kupo verifies** against the spec's acceptance checks → emits `verify.envelope.json` (ECL INFORM carrying verify_pass) with `from.eidolon = kupo` (distinct from maker `vivi`).
7. **ESL verify gate** (conformance checker or `eidolons run --verify`) blocks if maker==checker; passes if distinct.
8. **Status transitions to `verified`**.
9. **Drift-check transition** (same verifier as step 6) re-derives acceptance vs CRYSTALIUM Semantic spec + actual tree → no drift → `drift_checked = true`.
10. **IDG archives** the change → `status = archived` + emits `archive.envelope.json` (ECL ACKNOWLEDGE) → the spec is promoted to CRYSTALIUM Semantic layer via `mcp__crystalium__crystalium_commit`.
11. **Living-spec-of-record** is now available for later changes to query and reference (bi-temporal updates if future changes alter behavior).

---

## Provenance

| Claim | Source | Confidence |
|-------|--------|-----------|
| EIIS/ECL sibling-repo shape + composition works | ATLAS scout §6 + CLAUDE.md ECL block + live nexus codebase | H (direct) |
| ESL uses ECL performative set (closed 10, no extensions) | ECL spec/esl-1.0.md §7 | H (spec reference) |
| CRYSTALIUM Semantic/Execution/bi-temporal primitives exist | CRYSTALIUM codebase + CLAUDE.md memory block | H (direct) |
| Maker/checker enforced on ECL verify envelope | spec/esl-1.0.md §5 + ECL verify_pass performative | H (spec + code) |
| Independent evolution of three contracts is feasible | FORGE decision reasoning (C1/C5/C7) | M (design analysis) |

