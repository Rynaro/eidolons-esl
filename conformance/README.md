# ESL conformance checker

Standalone bash 3.2 conformance checker for ESL v1.0. Deterministic — no LLM,
no network. No runtime dependency on the nexus or any Eidolon repo.

## Hard requirements

- `bash` 3.2+ (the macOS system shell)
- `jq` (any reasonably modern version)
- POSIX coreutils (`find`, `sort`, `sed`, `basename`)

## Usage

```bash
# Validate a change folder (warn mode — report, exit 0):
bash conformance/esl-conformance.sh .spectra/changes/my-change/

# Block mode — exit 3 on any hard violation (use in CI / verify gates):
bash conformance/esl-conformance.sh .spectra/changes/my-change/ --mode block

# Machine-readable summary on stdout (findings still go to stderr):
bash conformance/esl-conformance.sh .spectra/changes/my-change/ --mode block --json
```

## Exit codes (ESL v1.0 §8.3)

| Code | Meaning |
|---|---|
| 0 | Conformant, or warnings-only under `--mode warn` (the default). |
| 1 | Usage error — bad args, missing/non-directory change folder, missing `jq`. |
| 3 | A hard violation under `--mode block`. |

`2` is RESERVED.

## The mechanical checks (six MUST + one SHOULD advisory)

| ID | Level | Asserts |
|---|---|---|
| C1 | MUST | `change.json` is present and valid JSON |
| C2a | MUST | `status` is a legal enum value (proposed/deliberated/in_progress/verified/archived) |
| C2b | MUST | `tier` is a legal enum value (trivial/lite/full) |
| C3 | MUST | tier-appropriate artifacts: trivial→no spec required; lite→one-page `spec.md` + non-empty `acceptance_checks`; full→`spec.{md,yaml}` |
| C4 | MUST | maker ≠ checker (and verify-envelope `from.eidolon` ≠ maker) when `status` ∈ {verified, archived} |
| C5 | MUST | `drift_checked == true` before `status == archived` |
| C6 | MUST | any `*.envelope.json` sidecar is well-formed JSON with `performative` in the ECL closed-10 set |
| C7 | SHOULD | **(advisory, never blocks)** any `acceptance_checks[]` item in the EARS form (an object declaring ≥1 of `given`/`when`/`then`) carries all of `given`/`when`/`then`/`verify_method` |

**C7 is advisory.** A C7-only failure does NOT change the exit code — it stays
`0` even under `--mode block` (only the MUST checks C1–C6 block). Plain-string
and minimal `{id, verify_method}` `acceptance_checks` items produce **no** C7
finding; the optional EARS form (ESL §2.5) is opt-in polish.

The ECL performative set is REFERENCED from a vendored constant in the script,
pointing at `schemas/performative.v1.json` in `Rynaro/eidolons-ecl`. ESL does not
re-enumerate it as ESL-owned. The checker does NOT verify ECL SHA-256 integrity —
that is the ECL verify gate's job (`eidolons run --verify`).

## Fixtures (exit-code contract)

Under `tests/`:

| Fixture | Mode | Expected | Proves |
|---|---|---|---|
| `maker-equals-checker/` | block | 3 | C4 — self-verify blocked |
| `archive-no-drift/` | block | 3 | C5 — drift-before-archive enforced |
| `full-missing-spec/` | block | 3 | C3 — full tier requires `spec.{md,yaml}` |
| `lite-missing-spec/` | block | 3 | C3 — lite tier requires one-page `spec.md` |
| `trivial-no-spec/` | block | 0 | C3 — trivial without a spec is NOT a violation |
| `ears-complete/` | block | 0 | C7 — EARS-complete item → C7 ok (advisory) |
| `ears-missing-field/` | block | 0 | C7 — EARS item missing `then` → C7 **fail** but exit 0 (advisory; only C1–C6 block) |

The conformant tier examples under `../examples/` (including the EARS-form
`lite-ears-complete/`) exit 0 in block mode.
