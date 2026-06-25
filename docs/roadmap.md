# Roadmap: The Next Wave (v1.1+)

**Version:** 1.0 prospective (ESL v1.0 delivered; v1.1+ candidates named)  
**Date:** 2026-06-24

---

## Philosophy

ESL v1.0 is a **minimal, thin, correct contract** — a coordination grammar over existing Eidolons. The roadmap names candidates for v1.1 and beyond, each with a one-line rationale and a confidence level. None is in scope for v1.0.

**Gating principle:** "Structure must earn its keep." Every candidate must solve a real problem or close an evidence gap. Ceremonial additions are rejected.

---

## v1.1 Candidates (high confidence — 0.75+)

### 1. `eidolons spec` CLI verb (optional; defer unless demanded)

**Rationale:** The grammar is already driven by existing Eidolons (SPECTRA emits, Vivi implements, Kupo verifies) + `eidolons run --verify` + CRYSTALIUM. A CLI verb that wraps this is **unfalsified ceremony** until a real consumer demands it (FORGE principle: no verb without a use case).

**What it would do:**
- `eidolons spec init <change_id>` — scaffold a new change folder from the appropriate tier template.
- `eidolons spec status <change_id>` — print the current state and any conformance warnings.
- `eidolons spec archive <change_id>` — move the folder to `archive/[date]-<change_id>/` and promote to CRYSTALIUM Semantic.

**Evidence gap:** zero consumer requests as of H1 2026. **Decision:** defer to v1.1 pending real adoption signals.

**Implementation note (if revisited):** **dual-allowlist gotcha** — a new `eidolons spec` verb MUST be registered in BOTH:
- `cli/eidolons` (the outer case statement, line ~265 for `mcp`)
- `cli/src/spec.sh` (the inner sub-dispatcher, like `cli/src/mcp.sh`)

A verb registered in one place only is inert. Confirmed for `mcp` (cli/eidolons:265 + cli/src/mcp.sh:49-77) and `telemetry`. This is a P0 checklist item for any new verb.

**Confidence:** 0.70 (clear design, low cost, but no demand signal yet).

---

### 2. Per-Eidolon ESL adoption (SPECTRA, FORGE, Vivi, Kupo, VIGIL, IDG each wire their lifecycle role)

**Rationale:** ESL v1.0 defines the **contract** (what state transitions mean, when drift-check runs); each Eidolon MUST wire its role into that contract (SPECTRA emits `proposed`, Vivi handles `in_progress`, etc.). Today, only the contract is shipped; the Eidolons' per-v1.0 adoptions happen independently in their own repos.

**What it would mean:**
- **SPECTRA v4.9.1+** embeds ESL awareness: emits `change.json` with `status: proposed`, `tier`, `change_id` alongside `spec.{md,yaml}`. Routes through the right-sizing gate before emitting.
- **FORGE v1.0+** ships a critiquing Eidolon that understands the `deliberated` state, emits CRITIQUE + DECIDE envelopes.
- **Vivi v1.0+** understands `in_progress`, isolated worktree, maker identity carriage on the ECL envelope.
- **Kupo v1.0+** verifier role understands `verified` state, maker≠checker enforcement, drift-check re-derivation.
- **VIGIL v1.0+** understands escalation (verify_fail → ESCALATE when impl diverges from spec).
- **IDG v1.0+** understands `archived` state, CRYSTALIUM Semantic promotion, bi-temporal update on spec revision.

**Status:** Eidolons are already implementing this independently (SPECTRA v4.9.1 emits the artifacts; Vivi has loop-native impl). ESL v1.0 formalizes the contract; v1.1+ tracks per-Eidolon release notes as they adopt.

**Confidence:** 0.95 (near-certain; in progress).

---

### 3. Native baseline eval (Eidolons-native SDD on real consumer projects; close GAP-B)

**Rationale:** All six failure-mode evidence comes from external tools (Spec Kit, Kiro, OpenSpec). ESL + Eidolons have never been measured on a real project. The first consumer adoption IS the de-facto baseline (GAP-B in FORGE).

**What it would mean:**
- A real consumer project (>10K LOC, team of 2–4, real deadlines) adopts ESL + the Eidolons for a quarter.
- Measure: defect rate, delivery velocity, spec adherence degradation (drift-check re-derivation success rate), team cognitive load (survey).
- Compare: pre-ESL ad-hoc vs. ESL-structured on identical task domains.
- Output: a technical report grounding the "structure must earn its keep" principle with Eidolons-native data.

**Status:** Not yet done. Owner/nexus maintainers can propose a candidate project.

**Confidence:** 0.60 (depends on finding a volunteer project; medium effort).

---

### 4. EARS-style acceptance-check helpers (optional; polish)

**Rationale:** SPECTRA's GIVEN/WHEN/THEN notation is close to EARS (Event-driven Requirements Specification, AWS Kiro standard: `WHEN [event] THE SYSTEM SHALL [action]`). ESL could optionally provide a Markdown linter or template that enforces EARS-style `acceptance_checks[]` format for consistency.

**What it would do:**
- Optional schema for `acceptance_checks[]` items: `{"id": "...", "given": "...", "when": "...", "then": "...", "verify_method": "..."}`
- A linter that flags malformed GIVEN/WHEN/THEN blocks in `spec.md`.
- A template generator that scaffolds EARS-format checks from a feature description.

**Status:** Deferred for v1.1; would be a cosmetic improvement only.

**Confidence:** 0.50 (nice-to-have; low priority).

---

### 5. Junction-over-bus spec dispatch demo (reference impl; polish)

**Rationale:** ESL defines that ECL envelopes CAN ride Junction's `plan_dispatch` (wiring_mode: transport). A worked example (or reference shell script) that dispatches a change proposal over the bus (instead of on-disk) would demonstrate the integration.

**What it would be:**
- A reference shell script: `.spectra/changes/<change_id>/dispatch.sh` that:
  - Reads `change.json` + `spec.{md,yaml}`
  - Serializes as an ECL envelope
  - Dispatches to Eidolons via `mcp__junction__plan_dispatch`
  - Receives responses, updates `change.json` status
- A worked example in `examples/full-new-subsystem/` showing both on-disk (fallback) and Junction dispatch paths.

**Status:** Deferred for v1.1; a nice-to-have integration demo.

**Confidence:** 0.65 (clear design, moderate effort, useful demo).

---

## v1.2 Candidates (medium confidence — 0.50–0.74)

### 6. Release/vendor tarball (if ESL becomes a roster-consumed asset)

**Rationale:** Today ESL is installed via `git clone`. If downstream projects want to pin ESL as a dependency (like they pin `eidolons-ecl`/`eidolons-eiis`), a vendor tarball + SHA-256 release asset would enable binary reproducibility checks.

**What it would mean:**
- GitHub release asset: `eidolons-esl-1.0.0.tar.gz` (spec + schema + conformance checker + templates + examples)
- Parallel `eidolons.lock` entry in the nexus: `esl@1.0` pinning the tarball SHA.
- Nexus integration: `eidolons upgrade self` pulls the latest ESL tarball and merges it into local `.esl/` or equivalent.

**Status:** Depends on nexus maintainers' decision to roster-pin contracts. Not urgent.

**Confidence:** 0.60 (clear design; depends on nexus adoption decision).

---

## Carried GAPs (not roadmap items; ownership clarification)

| Gap | What it is | Who closes it | When |
|---|---|---|---|
| **GAP-A** | Maker/checker split is endorsed, not T1-proven | A native Eidolons eval showing lift or no lift | v1.1 (post-GAP-B eval) |
| **GAP-B** | No Eidolons-native SDD baseline; all evidence is external | A real consumer project 1-quarter trial + report | v1.1 or later |
| **GAP-C** | Repo name `eidolons-esl` + stamp `ESL_VERSION` ratification | Owner | ✅ **RESOLVED 2026-06-24** (owner ratified) |

---

## Rejected v1.1 Candidates (anti-roadmap)

| Candidate | Rejection reason | Rationale |
|---|---|---|
| **A standalone `eidolons-esl` runtime Eidolon** | The grammar is already implemented by existing Eidolons; building an 8th Eidolon would violate "structure must earn its keep." | FORGE decision: reuse, don't reinvent. |
| **Re-declaring SPECTRA spec schema into change.json** | Violates the anti-scope clause (P0 contract violation); creates schema drift between ESL and SPECTRA. | Anti-pattern explicitly forbidden. |
| **A top-level `.esl/` directory instead of `.spectra/changes/`** | Breaks SPECTRA's "everything under `.spectra/`" convention; duplicates directory structure. | Maintain hierarchy cohesion. |
| **Optional acceptance-check verification in the conformance checker** | Adds LLM-dependent behavior to a deterministic, offline bash script. | Mechanical checker, no network/LLM. |
| **Automatic drift-check repair** | Verifier cannot edit (P0 rule: maker≠checker means checker never modifies payload). | Design invariant. |

---

## v1.0 Acceptance Gates — all PASSED (2026-06-25)

v1.0 shipped against these mechanical gates; every one is green in CI and was orchestrator-verified at build time:

| Gate | Criterion | Owner |
|---|---|---|
| **G1** | `schema/change.v1.json` is valid JSON (`jq empty` exits 0) | Vivi (build) |
| **G2** | All three tier examples conform (`conformance/esl-conformance.sh --mode block` exits 0) | Vivi (build) |
| **G3** | Maker==checker is blocked (RED-first fixture → exit 3) | Vivi (build) |
| **G4** | Drift-check-before-archive enforced (RED-first fixture → exit 3) | Vivi (build) |
| **G5** | Tier-required artifacts enforced (trivial: no spec OK; lite/full: spec REQUIRED) | Vivi (build) |
| **G6** | Conformance checker is bash 3.2 (shellcheck clean + no 4+ constructs) | Vivi (build) |
| **G7** | Checker is deterministic + offline (two runs = byte-identical JSON; no network) | Vivi (build) |
| **G8** | Anti-scope clause holds (no re-declared schemas/performatives/layers; all cross-refs are pointers) | Vivi (build) |
| **G9** | Self-discipline: `spec/esl-1.0.md` ≤~330 lines; README ≤~180 | Vivi (build) |
| **G10** | Exit-code contract (0=conform, 1=usage-error, 3=block-violation) | Vivi (build) |
| **G11** | Nexus-awareness consistent (CLAUDE.md + EIDOLONS.md blocks; no `eidolons spec` verb in v1.0) | Nexus PR [#378](https://github.com/Rynaro/eidolons/pull/378) — open |

---

## Timeline Sketch (provisional; owner-driven)

| Phase | Target | Deliverables |
|---|---|---|
| **v1.0** | **SHIPPED 2026-06-25** | Spec LOCKED; repo built + published ([`Rynaro/eidolons-esl`](https://github.com/Rynaro/eidolons-esl), gates G1–G10 green); nexus-awareness PR [#378](https://github.com/Rynaro/eidolons/pull/378) open; name ratified. |
| **v1.1** | 2026-09 | Per-Eidolon adoptions finalized. CLI verb (optional, if demanded). Native baseline eval (GAP-B closure). Roadmap-2 candidates scoped. |
| **v1.2** | 2026-12+ | Release tarball (if roster-pinned). EARS helpers (polish). Junction demo. Further GAP closure. |

---

## Provenance

| Claim | Source | Confidence |
|-------|--------|-----------|
| Six failure modes CONFIRMED | sdd-research.md + Thoughtworks Tech Radar | T1/T2 |
| Right-sizing gate defends over-specification | spec/esl-1.0.md §4 + stress-test | M (design) |
| Dual-allowlist gotcha on new CLI verbs | CLAUDE.md feedback + atlas-sdd-scout.md | H (direct) |
| Living-spec drift-check ~90% recovery | SLUMP arXiv 2603.17104 | T1 |
| Per-Eidolon adoption in progress | SPECTRA v4.9.1, Vivi v1.0, Kupo v1.0 release notes | H (direct) |
| Structure-must-earn-its-keep principle | FORGE decision + sdd-research.md §1.5 (E2) | T1/M |

