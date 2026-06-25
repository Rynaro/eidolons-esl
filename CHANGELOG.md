# Changelog

All notable changes to ESL (Eidolons Spec Lifecycle) are documented in this
file. Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [SemVer 2.0](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **`docs/escalation.md`** — formalizes the opt-in → forced enforcement policy
  (FORGE Decision 3): `--mode warn` is the advisory default; a project escalates
  to `--mode block` when any project-aggregate of the §4.2 right-sizing signals
  trips a mechanical threshold (`change_count ≥ N`, `repo_loc ≥ L`, or
  `full_ratio ≥ R`; seed defaults N=10 / L=50_000 / R=0.4, tunable knobs). The
  flip is recorded **nexus-side** in `eidolons.mcp.lock`, NOT in `esl-1.0.md`
  (ESL stays opt-in per §1.4); tonberry ships the assessment + the lever while
  the auto-flip *recording* is a deferred nexus follow-up.
- **`spec/esl-1.0.md` §8.3** — a non-normative NOTE pointing to
  `docs/escalation.md`. No change to the opt-in P0, the conformance checks, the
  schema, or the exit-code contract — escalation is governance, not a C-check.

## [1.0.0] — 2026-06-24

### Added

- **`spec/esl-1.0.md`** — the normative Spec Lifecycle contract. Nine sections:
  §1 scope & anti-scope (OWNS/DEFERS by version; the anti-scope MUST-NOT); §2 the
  change unit (`change_id`, fields by reference to `schema/change.v1.json`); §3
  the five-state, tier-skippable lifecycle state machine; §4 the mandatory
  mechanical right-sizing gate (three observable signals → trivial/lite/full);
  §5 the maker ≠ checker rule; §6 living-spec / drift (CRYSTALIUM
  Execution/Semantic/bi-temporal `update` + the `drift_check` transition); §7 the
  ECL closed-10 performative mapping (no ACCEPT; ingest rides `context_delta`);
  §8 conformance (exit codes, warn/block, RFC-2119); §9 versioning.
- **`schema/change.v1.json`** — vendorable JSON Schema (draft 2020-12) for the
  per-change `change.json` manifest. REQUIRED `esl_version`, `change_id`,
  `status`, `tier`, `maker`, `checker`, `acceptance_checks[]`, `spec_ref`;
  OPTIONAL `supersedes`, `superseded_by`, `created_at`, `drift_checked`,
  `archive_path`. The `status` and `tier` enums are the only ESL-owned schema.
- **`conformance/esl-conformance.sh`** — standalone bash 3.2 deterministic
  checker (no LLM, no network). Six mechanical checks (valid JSON; legal
  status/tier enums; tier-appropriate artifacts; maker ≠ checker at
  verified/archived; drift_checked before archive; ECL envelope well-formedness
  + performative-in-closed-10). Modes `--mode warn` (default) / `--mode block`;
  exit codes `0`/`1`/`3`; `--json` machine summary on stdout.
- **`conformance/README.md`** — exit-code table, usage, and the six-check table.
- **`conformance/tests/`** — five violating/control fixtures proving C3/C4/C5 in
  block mode (`maker-equals-checker`, `archive-no-drift`, `full-missing-spec`,
  `lite-missing-spec`, `trivial-no-spec`).
- **`templates/{trivial,lite,full}/`** — per-tier change-folder skeletons. The
  `lite`/`full` spec stubs carry the anti-scope header (SPECTRA owns the spec
  schema; the stub is a slot).
- **`examples/{trivial-typo-fix,lite-add-flag,full-new-subsystem}/`** — one
  worked change per tier, each with `change.json` + ECL `.envelope.json`
  sidecars. The lite/full examples demonstrate maker ≠ checker; the full example
  demonstrates the archive snapshot + Semantic-promotion note.
- **`ESL_VERSION`** — root stamp `1.0` (mirrors `ECL_VERSION` / `EIIS_VERSION`).

### Notes

- ESL is **opt-in** for v1.0; live ECL/EIIS consumers remain conformant
  unchanged.
- The anti-scope clause is P0: ESL names and references SPECTRA/ECL/CRYSTALIUM
  shapes by version and never re-declares them.
- The nexus-awareness PR (the CLAUDE.md ESL block + the EIDOLONS.md cortex
  pointer) is a **separate** follow-on in the nexus repo; no `eidolons spec` CLI
  verb ships in v1.0.

[Unreleased]: https://github.com/Rynaro/eidolons-esl/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/Rynaro/eidolons-esl/releases/tag/v1.0.0
