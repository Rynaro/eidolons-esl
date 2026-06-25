# Adoption: How to Use ESL in Your Project

**Version:** 1.0  
**Date:** 2026-06-24

---

## Before You Start: Prerequisites

Your consumer project should already have:

1. **Eidolons installed** via `eidolons init --preset full` or equivalent.
   - SPECTRA (spec author)
   - FORGE (deliberation)
   - Vivi (loop-native implementer)
   - Kupo (verifier)
   - VIGIL (debugger, for failure escalation)
   - IDG (documentation)

2. **ECL v1.0 awareness** — your `.eidolons/cortex/EIDOLONS.md` includes the ECL block, and `eidolons run --verify` is available.

3. **CRYSTALIUM v1.4+ memory** — installed for Semantic layer promotion (living-spec storage).

4. **A `.spectra/` root** — SPECTRA's output directory for plans, state, logs, and (new) changes.

If any of these are missing, adopt them first.

---

## The Change Folder Layout

A change lives in your consumer project under `.spectra/changes/<change_id>/`, a per-change folder following OpenSpec convention:

```
.spectra/
├── plans/            # existing — SPECTRA per-feature spec outputs
├── state/            # existing
├── logs/             # existing
└── changes/          # NEW (ESL) — per-change lifecycle folders
    ├── feature-auth-redesign/
    │   ├── change.json                    # the ESL manifest
    │   ├── spec.md                        # tier-dependent (lite/full)
    │   ├── spec.yaml                      # tier-dependent (full only)
    │   ├── propose.envelope.json          # ECL PROPOSE sidecar
    │   ├── critique.envelope.json         # ECL CRITIQUE (deliberated only)
    │   ├── verify.envelope.json           # ECL INFORM(verify_pass) sidecar
    │   └── archive/
    │       └── 2026-06-24-feature-auth-redesign/
    │           ├── change.json            # snapshot on archive
    │           ├── spec.md
    │           └── spec.yaml
    └── fix-typo-in-readme/
        ├── change.json
        └── delegate.envelope.json         # trivial route: DELEGATE → Kupo only
```

**Key:** on `archived`, the entire change folder content is moved under `archive/[date]-<change_id>/` so the active `.spectra/changes/` surface stays small (the history is auditable on disk; the spec-of-record lives in CRYSTALIUM Semantic layer).

---

## Starting a Change: The Right-Sizing Gate

Every change entry starts at the `proposed` state. Before writing any spec, a **mandatory mechanical right-sizing gate** classifies your change into one of three tiers.

### The three signals (observable, never discretionary)

1. **Files-touched estimate** — how many files will this change touch? (a number from your proposal)
2. **SPECTRA /12 complexity score** — run SPECTRA's complexity rubric on your feature description; it returns a score 1–12.
3. **Trade-off-present boolean** — is there a real competing-approach decision (multiple valid routes)? (true/false)

### The routing decision

**Trivial tier** (all conditions must hold):
- `files-touched ≤ 2` **AND**
- `SPECTRA /12 score ≤ 4` **AND**
- No new behavior contract (bug fix, typo, refactor with no API change)

→ **Route to Kupo.** Kupo emits a no-spec micro-change. Status → `proposed` (no deliberation, no impl planning) → `in_progress` → verified by Kupo verifier (distinct from Kupo executor) → `archived`. **No spec.{md,yaml} required.** `spec_ref` may be `null`.

**Lite tier** (all conditions must hold):
- Bounded scope (5–6 files touched)
- `SPECTRA /12 score 5–6`
- Single behavior (one cohesive feature, no subsystem redesign)
- No genuine trade-off

→ **SPECTRA emits a one-page spec** (GIVEN/WHEN/THEN + acceptance_checks only — no deliberation section, no plan/tasks split). Status → `proposed` → `in_progress` (skip `deliberated`) → verified → `archived`. **One-page spec.md REQUIRED.** conformance checker blocks if absent.

**Full tier** (any condition suffices):
- `SPECTRA /12 score ≥ 7` **OR**
- A genuine trade-off exists (competing approaches, architectural decision) **OR**
- System-wide blast radius (change affects multiple subsystems)

→ **Full lifecycle.** Status → `proposed` → `deliberated` (FORGE critiques, decides routing) → `in_progress` → verified → `archived`. **Spec.{md,yaml} REQUIRED** (full SPECTRA format). Deliberation cannot be skipped.

**Example:**
- "Add a login email field to the auth schema" → files ≤ 2, /12 = 3, no trade-off → **trivial** (Kupo, no spec).
- "Add a new OAuth provider with fallback logic" → files ≤ 5, /12 = 5, single provider → **lite** (one-page spec, skip deliberation).
- "Redesign the entire auth subsystem to support multi-tenant isolation" → /12 = 9, multiple architectural routes → **full** (all states, FORGE deliberates).

---

## Tier-Specific Workflows

### Trivial (Kupo micro-fix, no spec)

**Starter template:** `templates/trivial/change.json`

```json
{
  "esl_version": "1.0",
  "change_id": "fix-typo-in-readme",
  "status": "proposed",
  "tier": "trivial",
  "maker": "kupo",
  "checker": "vigil",
  "acceptance_checks": [
    {
      "id": "no-additional-typos-introduced",
      "verify_method": "manual-scan"
    }
  ],
  "spec_ref": null,
  "created_at": "2026-06-24T10:00:00Z"
}
```

1. Create the folder `.spectra/changes/fix-typo-in-readme/`.
2. Copy `change.json` into it.
3. Emit `delegate.envelope.json` (ECL PROPOSE performative, but routed directly to Kupo).
4. Kupo makes the fix + emits verify sidecar (with `from.eidolon = kupo-verifier`, distinct from maker).
5. Conformance checker runs: `bash /path/to/esl-conformance.sh .spectra/changes/fix-typo-in-readme/ --mode block` → exits 0.
6. Status → `verified` → `archived` (IDG moves the folder to `archive/[date]-fix-typo-in-readme/`).
7. Conformance on archived folder → checks `drift_checked: true`, exits 0.

### Lite (one-page spec, skip deliberation)

**Starter template:** `templates/lite/change.json` + `templates/lite/spec.md`

1. SPECTRA emits a **one-page spec.md** containing only:
   - **GIVEN/WHEN/THEN narrative** (one or two scenarios)
   - **acceptance_checks** array (3–5 checks, not 20+)
   - No deliberation section, no plan/tasks split
2. Create `.spectra/changes/add-oauth-provider/`.
3. Copy `change.json` (status `proposed`, tier `lite`) and the one-page `spec.md`.
4. Emit `propose.envelope.json` (ECL PROPOSE).
5. Status → `proposed` (no deliberation triggered because tier = `lite`).
6. Vivi implements in an isolated worktree.
7. Emit `verify.envelope.json` (ECL INFORM carrying verify_pass) with verifier identity distinct from Vivi.
8. Conformance checker validates: `bash esl-conformance.sh .spectra/changes/add-oauth-provider/ --mode block` → exits 0 if spec.md is present and non-empty.
9. Status → `verified` → drift-check (verifier re-derives acceptance checks) → `archived`.

### Full (all five states, FORGE deliberation)

**Starter template:** `templates/full/change.json` + `templates/full/spec.{md,yaml}`

1. SPECTRA emits **spec.md + spec.yaml** (full SPECTRA format).
2. Create `.spectra/changes/redesign-auth-subsystem/`.
3. Copy templates, fill placeholders.
4. Emit `propose.envelope.json` (ECL PROPOSE).
5. Status → `proposed`.
6. **FORGE deliberation** (because tier = `full`):
   - FORGE critiques the proposed approach, names competing options.
   - Emit `critique.envelope.json` (ECL CRITIQUE) naming the objections.
   - Emit revised `propose.envelope.json` (ECL PROPOSE) with new routing.
   - Emit `decide.envelope.json` (ECL DECIDE) recording the chosen architecture.
7. Status → `deliberated`.
8. Vivi implements (isolated worktree).
9. Emit `delegate.envelope.json` + code.
10. Status → `in_progress`.
11. Kupo verifier or VIGIL verifies; emit `verify.envelope.json` (verifier ≠ maker).
12. Status → `verified`.
13. **Drift-check transition** (same verifier):
    - Re-derive the acceptance checks (from spec.yaml GIVEN/WHEN/THEN) against the actual implemented tree.
    - If divergence found → emit `verify_fail` (ECL ESCALATE) → status returns to `in_progress`.
    - If no divergence → set `drift_checked: true`.
14. Status → `archived`; IDG moves folder to `archive/[date]-redesign-auth-subsystem/`; the spec is promoted to CRYSTALIUM Semantic layer.

---

## Conformance Checking: Mechanical Verification

### Quick start

```bash
# Clone ESL (if not already available)
git clone https://github.com/Rynaro/eidolons-esl /tmp/esl

# Check a change folder (warn mode — default)
bash /tmp/esl/conformance/esl-conformance.sh .spectra/changes/my-change/

# Block mode (fails hard on violations; use in CI/verify gates)
bash /tmp/esl/conformance/esl-conformance.sh .spectra/changes/my-change/ --mode block

# JSON machine output (for automation)
bash /tmp/esl/conformance/esl-conformance.sh .spectra/changes/my-change/ --json
```

### Exit codes

- **0** — conformant (or warnings only in `--mode warn`)
- **1** — usage error (bad args, missing folder)
- **3** — hard violation in `--mode block` (e.g., maker==checker, drift_check missing before archive, illegal status, missing tier-required artifact)

### What the checker validates

1. `change.json` exists and is valid JSON (`jq empty` passes).
2. `status` is one of {`proposed`, `deliberated`, `in_progress`, `verified`, `archived`}; `tier` is one of {`trivial`, `lite`, `full`}.
3. **Tier-appropriate artifacts are present:**
   - `trivial` → no `spec.{md,yaml}` required (spec_ref may be null).
   - `lite` → one-page `spec.md` present with non-empty `acceptance_checks`.
   - `full` → both `spec.md` and `spec.yaml` present.
4. **Maker ≠ checker** when status ∈ {`verified`, `archived`} (reads `from.eidolon` from the verify envelope sidecar).
5. **`drift_checked: true`** before status = `archived` (enforced; missing drift-check blocks promotion).
6. **ECL envelope sidecars** are well-formed JSON with `performative` in the closed 10-set (PROPOSE, CRITIQUE, DECIDE, DELEGATE, ACKNOWLEDGE, INFORM, ESCALATE, REFUSE, RESUME, ACCEPT).

---

## The Maker ≠ Checker Rule (P0)

The most important invariant: **a change cannot reach `verified` or `archived` if the maker and checker are the same identity.**

**Mechanically enforced:**
- Vivi (maker) emits code + an ECL `verify` block containing the SHA-256 of the changed payload.
- Kupo verifier or VIGIL (distinct identity) re-derives the SHA-256 against the spec's acceptance checks.
- The `verify_pass` envelope carries `from.eidolon = kupo` (or `vigil`), distinct from maker `vivi`.
- **Conformance checker blocks if `from.eidolon == change.json.maker`.** This is checked both:
  - By `eidolons run --verify <envelope>` (at runtime, during verification).
  - By `conformance/esl-conformance.sh --mode block <folder>` (in CI, when archived).

If you see a `verify_pass` where the verifier is the same as the maker, the conformance checker exits 3 (hard block). The change cannot be promoted.

---

## Drift-Check: Living-Spec Verification

Before a change enters `archived`, a **drift-check transition** runs. The verifier (same identity as the one who verified in step 3) re-derives the acceptance checks against two sources:

1. **The CRYSTALIUM Semantic spec** (the promoted spec-of-record from prior changes).
2. **The actual implemented tree** (the code that Vivi wrote).

If the two diverge (the implementation outran the spec, or the spec is no longer accurate), the verifier emits `verify_fail` and the status returns to `in_progress` (not archived).

**Mechanically enforced:**
- Before `archived`, `change.json.drift_checked` MUST be `true`.
- Conformance checker blocks if `status == archived` but `drift_checked != true`.

**Why it matters:**
Research (SLUMP, arXiv 2603.17104, T1 March 2026) shows that living-spec drift-check recovers ~90% of the faithfulness gap between a spec and the actual implementation over multi-session projects. The drift-check is the primary defense against cognitive debt (failure mode #6).

---

## Living-Spec in CRYSTALIUM

When a change reaches `archived`, its spec is **promoted to CRYSTALIUM Semantic layer** — the durable spec-of-record. Unlike a one-shot prompt, it is:

1. **Versioned** — each change is a crystal entry with a `change_id`.
2. **Revisable** — if a later change alters behavior already in the Semantic spec, it uses CRYSTALIUM bi-temporal `update` (invalidate-old via `t_valid_to` + `superseded_by`, write-new, never hard-delete).
3. **Queryable** — later agents can recall the living spec via CRYSTALIUM `recall` (e.g., "what was the acceptance contract for the auth redesign?").

To promote, IDG (or your automation) calls:

```bash
mcp__crystalium__crystalium_commit \
  --layer semantic \
  --payload '{"summary":"Auth redesign complete","change_id":"redesign-auth-subsystem","spec_ref":"...",...}' \
  --provenance '{"source":"esl-change-archive","author_agent":"idg","created_at":"2026-06-24T..."}'
```

This is the Eidolon's responsibility, not ESL's — ESL only defines the transition. Vivi or IDG orchestrates the commit.

---

## Practical Scenario: Small Bug Fix → Trivial Route

1. A user reports a typo in the docs.
2. You propose: "Fix: capitalize 'Database' in ARCHITECTURE.md" → estimate 1 file, /12 = 2, no new behavior.
3. **Right-sizing gate** routes → `trivial`.
4. Create `.spectra/changes/fix-database-capitalization/change.json`:
   ```json
   {
     "esl_version": "1.0",
     "change_id": "fix-database-capitalization",
     "status": "proposed",
     "tier": "trivial",
     "maker": "kupo",
     "checker": "vigil",
     "acceptance_checks": [
       {"id": "database-capitalized", "verify_method": "grep-check"}
     ],
     "spec_ref": null
   }
   ```
5. Kupo makes the fix directly (no Vivi loop, no deliberation).
6. Kupo emits `delegate.envelope.json` (PROPOSE → Kupo route).
7. VIGIL verifies the fix (checks the grep pattern), emits `verify.envelope.json` with `from.eidolon: vigil`.
8. Status → `verified`.
9. Drift-check: VIGIL re-runs the grep against CRYSTALIUM Semantic spec (if this was a prior promise); no drift.
10. Status → `archived`; folder moves to `archive/2026-06-24-fix-database-capitalization/`.
11. Conformance: `bash esl-conformance.sh .spectra/changes/fix-database-capitalization/ --mode block` → exits 0.

**Time to change complete:** minutes (no ceremony, no backlog). That's the trivial route.

---

## Practical Scenario: New Feature → Lite Route

1. A user requests: "Add a reset-password email confirmation flow."
2. Estimate: 5 files (handler, template, test, doc, schema), /12 = 5, no competing architectures.
3. **Right-sizing gate** routes → `lite`.
4. SPECTRA emits a one-page spec.md:
   ```markdown
   # Reset-Password Email Confirmation

   **GIVEN** a user requests password reset
   **WHEN** the system sends a confirmation email with a time-limited link
   **THEN** only users with that valid link can change the password

   ## Acceptance Checks
   - Email is sent within 1 second of request
   - Link expires after 1 hour
   - Invalid links return 403 Forbidden
   ```
5. Create `.spectra/changes/add-reset-password/change.json` (tier `lite`, status `proposed`).
6. Emit `propose.envelope.json`.
7. Status → `proposed` (no deliberation, lite tier skips FORGE).
8. Vivi implements in a worktree → emits code + `verify.envelope.json` (verifier ≠ Vivi).
9. Kupo verifier checks the three acceptance checks; emit `verify_pass`.
10. Status → `verified` → drift-check (verifier re-derives checks against the tree) → `archived`.

**Time to change complete:** hours (minimal spec ceremony, full verification).

---

## Provenance

| Claim | Source | Confidence |
|-------|--------|-----------|
| Change folder layout under `.spectra/changes/` | spec/esl-1.0.md §2/§9; SPECTRA `.spectra/` convention | H (spec + precedent) |
| Right-sizing gate signals and routing | spec/esl-1.0.md §4; PRODUCT-BRIEF failure-mode defense | H (spec) |
| Per-tier lifecycle paths (trivial/lite/full) | spec/esl-1.0.md §3/§4; worked examples | H (spec + examples) |
| Maker/checker mechanized via ECL verify envelope | spec/esl-1.0.md §5; ECL spec performative-mapping | H (spec) |
| Drift-check before archive, SLUMP ~90% recovery | spec/esl-1.0.md §6; arXiv 2603.17104 | T1 (direct) |
| Conformance checker deterministic, bash 3.2 | conformance/esl-conformance.sh + README | H (direct) |
| CRYSTALIUM Semantic promotion, bi-temporal update | CLAUDE.md memory block + nexus codebase | H (direct) |

