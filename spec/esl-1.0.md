# ESL — Eidolons Spec Lifecycle, v1.0

**Status:** stable · **Stamp:** `ESL_VERSION` = `1.0` · **Sibling of:** ECL, EIIS

The key words MUST, MUST NOT, REQUIRED, SHALL, SHOULD, SHOULD NOT, MAY are to be
interpreted as described in RFC 2119 / RFC 8174 (BCP 14).

ESL is a thin **coordination grammar** over the existing Eidolons. It is a
specification plus a standalone bash conformance checker — NOT a runtime, NOT a
roster Eidolon, NOT an install standard. The host LLM remains the runtime; ECL
remains the transport; SPECTRA remains the spec author; CRYSTALIUM remains the
memory.

---

## §1 Scope & anti-scope

1.1 ESL **OWNS** exactly four things: (a) the lifecycle **status vocabulary +
transitions** (§3); (b) the **`change_id` / supersede / diff grammar** (§2, §9);
(c) the mandatory **right-sizing gate** (§4); (d) the **`drift_check` transition
+ spec-of-record rule** (§6).

1.2 ESL **DEFERS** everything else, each named BY VERSION and never re-declared:
artifact schema → **SPECTRA** `spec.{md,yaml}`; transport → **ECL v1.0** envelopes
+ SHA-256 verify gate; persistence → **CRYSTALIUM** bi-temporal `update` +
Semantic/Execution layers; bus → **Junction** `plan_dispatch`.

1.3 **Anti-scope clause (P0).** A conforming ESL artifact MAY **name** lifecycle
fields and **reference** a deferred schema, performative, or memory layer BY
VERSION. It MUST NOT **re-declare** an artifact schema, a performative, or a
memory layer. Re-declaration is a contract bug. The ONLY ESL-owned schema is the
`status` and `tier` enums in `schema/change.v1.json`.

1.4 ESL is **opt-in** for v1.0. A consumer with no ESL awareness remains
conformant to ECL/EIIS unchanged.

## §2 The change unit

2.1 The unit of work is a **change**, keyed by a `change_id`. Each change is a
per-change folder (OpenSpec-style), described in §9.

2.2 Every change folder MUST contain a `change.json` manifest. Its fields are
defined by `schema/change.v1.json` and are REFERENCED here, not re-declared:
`esl_version`, `change_id`, `status`, `tier`, `maker`, `checker`,
`acceptance_checks[]`, `spec_ref` (REQUIRED); `supersedes`, `superseded_by`,
`created_at`, `drift_checked`, `archive_path` (OPTIONAL).

2.3 `spec_ref` MUST be a relative path to the SPECTRA-owned `spec.{md,yaml}` or
`null`. It MUST NOT inline the SPECTRA schema.

2.4 `acceptance_checks[]` items carry a GIVEN/WHEN/THEN `id` that points INTO
`spec_ref` plus an OPTIONAL `verify_method` reference. ESL MUST NOT re-define
SPECTRA's GIVEN/WHEN/THEN format.

## §3 Lifecycle state machine

3.1 A change moves through five states. Every transition **after `proposed` is
skippable per the right-sizing tier** (§4) — this is a right-sizable machine, not
a mandatory linear gate.

| # | State | Owner | Entering ECL performative | Skippable? |
|---|-------|-------|---------------------------|-----------|
| 0 | `proposed` | SPECTRA emits `spec.{md,yaml}`; Kupo emits a no-spec micro-change for trivial | `PROPOSE` | required entry |
| 1 | `deliberated` | FORGE (only on a real trade-off) | `CRITIQUE` → revised `PROPOSE`; `DECIDE` records routing/approval | **SKIP for lite/trivial** |
| 2 | `in_progress` | Vivi (loop-native maker, isolated worktree) | `DELEGATE` / `ACKNOWLEDGE` | required if ANY code |
| 3 | `verified` | distinct checker (Kupo verifier, or VIGIL on failure) — MUST NOT be the maker | `INFORM(verify_pass)`; `verify_fail` → `ESCALATE` | required to promote |
| 4 | `archived` | IDG moves the folder to `archive/<date>-<change_id>/`; living spec promoted to CRYSTALIUM Semantic | `ACKNOWLEDGE`; `INFORM(promotion)` | required exit |

3.2 The only mandatory spine is `proposed` → (`in_progress` if any code) →
`verified` → `archived`. `deliberated` is conditional on a real trade-off; the
code-states are conditional on there being code.

3.3 `status` MUST be one of the five values above. A change MUST NOT enter
`archived` before satisfying §6.4.

## §4 Right-sizing gate

4.1 A **single mandatory, mechanical gate at `proposed` entry** classifies every
change into one of three tiers. Classification MUST NOT be skipped; its output is
usually "less."

4.2 The gate MUST be **mechanical** — a comparison on observable signals, never
an LLM "is this big?" judgment. It consumes three signals: (a) a **files-touched
estimate** (a number from the proposal); (b) the **SPECTRA /12 rubric score**
(already computed by SPECTRA's complexity matrix); (c) a **trade-off-present**
boolean (true iff a competing-approach decision exists — the FORGE trigger).

| Tier | Trigger (ALL must hold for the smaller tier) | Route | Path |
|------|-----------------------------------------------|-------|------|
| `trivial` | files ≤ 2 AND /12 ≤ 4 AND no new behavior contract | Kupo micro-fix, NO spec; `DELEGATE` → Kupo | bypass machine |
| `lite` | bounded scope AND /12 in 5–6 AND single behavior AND no trade-off | SPECTRA one-page spec (GIVEN/WHEN/THEN + acceptance_checks only) | `0→2→3→4` |
| `full` | /12 ≥ 7 OR trade-off present OR system-wide blast radius | full lifecycle | `0→1→2→3→4` |

4.3 **Routing precedence:** any `full` trigger wins; else `trivial` if all
trivial conditions hold; else `lite`. Given the same three signals, the gate MUST
yield the identical tier (determinism).

## §5 Maker/checker rule

5.1 Promotion to `verified` MUST require a `verify_pass` envelope whose author
identity (`from.eidolon`) is **distinct from** the maker identity
(`change.json.maker`). A self-verified change is a contract violation.

5.2 The mechanical lever is identity-inequality on the verify envelope, checkable
by `eidolons run --verify` (reusing `cli/src/verify_envelope.sh`) AND by
`conformance/esl-conformance.sh` (§8). When `status` ∈ {`verified`, `archived`},
the checker MUST block if `maker == checker` or if the verify envelope's
`from.eidolon == change.json.maker`.

5.3 The checker MUST NOT edit the payload. It emits `verify_pass` / `verify_fail`
only; `verify_fail` emits `ESCALATE` and returns the change to `in_progress`.

## §6 Living-spec / drift

6.1 Three CRYSTALIUM primitives do three distinct jobs. ESL REFERENCES these
layers by name and MUST NOT re-declare them.

| Job | CRYSTALIUM primitive (referenced) | When |
|---|---|---|
| In-flight spec (mutable) | Execution layer `plan_checkpoint` / `plan_replan` (TTL-bound; a re-scope is a `plan_replan` with `supersedes_id`) | `proposed`..`verified` |
| Durable spec-of-record | Semantic layer promotion (corroborated behavior contract) | verified + archived only |
| Revision of a promoted spec | bi-temporal `update` (invalidate-old via `t_valid_to` + `superseded_by`, write-new; never hard-delete) | a later change alters behavior |

6.2 Only verified + archived specs SHALL be promoted to the Semantic layer.

6.3 A later change that alters a behavior already in the Semantic spec MUST set
`change.json.supersedes` to the prior `change_id`; the revision MUST go through
bi-temporal `update` so the old revision stays auditable.

6.4 **`drift_check` transition.** Before `archived`, the same identity-distinct
checker from §5 MUST re-derive the change's `acceptance_checks` against the
promoted Semantic spec AND the actual tree. A mismatch MUST emit `verify_fail` →
`ESCALATE` (back to `in_progress`). `change.json.drift_checked` MUST be `true`
before a change may enter `archived`.

## §7 ECL performative mapping

7.1 ESL rides the **closed ECL v1.0 ten-performative set**. ESL REFERENCES that
set (see `schemas/performative.v1.json` in `Rynaro/eidolons-ecl`) and MUST NOT
enumerate it as ESL-owned.

7.2 Transition-to-performative mapping: entry = `PROPOSE`; deliberation =
`CRITIQUE` → revised `PROPOSE`, with `DECIDE` recording chosen routing/approval;
delegation = `DELEGATE`; acceptance = `ACKNOWLEDGE`; verification =
`INFORM(verify_pass)`, with `verify_fail` → `ESCALATE`; resume = `RESUME`;
refusal = `REFUSE`.

7.3 There is **NO ACCEPT performative** — acceptance is `ACKNOWLEDGE` or a return
`PROPOSE`. Memory ingest at each transition rides the envelope's `context_delta`
FIELD, not a performative.

## §8 Conformance

8.1 `conformance/esl-conformance.sh` is the normative checker: bash 3.2
compatible, deterministic (no LLM, no network), warn/block modes mirroring ECL.
Its input is a change-folder path.

8.2 It runs six mechanical checks: (1) `change.json` is valid JSON; (2) `status`
and `tier` are legal enum values; (3) tier-appropriate artifacts are present
(`trivial` → no spec required; `lite` → one-page `spec.md` with non-empty
`acceptance_checks`; `full` → `spec.{md,yaml}`); (4) maker ≠ checker when
`status` ∈ {`verified`, `archived`}; (5) `drift_checked == true` before
`archived`; (6) any ECL envelope sidecar is well-formed JSON with a
`performative` in the closed ten-set.

8.3 **Exit codes:** `0` conformant (or warnings only in `--mode warn`); `1` usage
error (bad args, missing folder); `3` a hard violation in `--mode block`. `2` is
RESERVED. All human-readable findings go to stderr; `--json` machine summary goes
to stdout only.

> **NOTE (governance, non-normative).** `--mode warn` is the advisory default; a
> project MAY escalate to `--mode block` on a mechanical project-aggregate of the
> §4.2 signals. That escalation is a nexus-side policy recorded in
> `eidolons.mcp.lock`, NOT here — ESL stays opt-in (§1.4). See `docs/escalation.md`.

## §9 Versioning

9.1 ESL versions at the document level (SemVer). The `ESL_VERSION` stamp declares
`MAJOR.MINOR`. A consumer reads it to decide whether ESL grammar is active,
mirroring `ECL_VERSION` / `EIIS_VERSION`.

9.2 A change folder lives under `.spectra/changes/<change_id>/`. It adds a
`changes/` sibling alongside SPECTRA's `plans/` / `state/` / `logs/`; it MUST NOT
introduce a new top-level consumer directory (preserving SPECTRA's
"everything under `.spectra/`" rule). On `archived`, the folder content is
snapshotted under `archive/<date>-<change_id>/`.

9.3 A version bump of a **referenced** schema (SPECTRA spec profile, ECL
envelope, CRYSTALIUM layer shape) is a `[REVERSAL-CONDITION]`, not a silent
follow: ESL MUST re-version its references deliberately rather than track an
upstream bump implicitly.
