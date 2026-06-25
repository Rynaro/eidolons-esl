# ESL — Eidolons Spec Lifecycle

[![conformance](https://github.com/Rynaro/eidolons-esl/actions/workflows/conformance.yml/badge.svg)](https://github.com/Rynaro/eidolons-esl/actions/workflows/conformance.yml)

ESL is the **spec-lifecycle coordination grammar** for Eidolons changes: a
right-sizable state machine, a mandatory mechanical right-sizing gate, and a
living-spec / drift contract. It is a plain-text standard plus a standalone
bash conformance checker.

ESL is a **sibling CONTRACT repo** — a peer of
[`Rynaro/eidolons-ecl`](https://github.com/Rynaro/eidolons-ecl) (ECL) and
`Rynaro/eidolons-eiis` (EIIS). It is **NOT** a roster Eidolon, **NOT** listed in
the nexus `roster/index.yaml`, and triggers **no** EIIS conformance check. Like
ECL/EIIS, it is made "aware" by a CLAUDE.md prose block, an EIDOLONS.md cortex
pointer, and a root `ESL_VERSION` stamp. ESL is **opt-in** for v1.0.

- **Spec:** [`spec/esl-1.0.md`](spec/esl-1.0.md) — the normative contract.
- **Schema:** [`schema/change.v1.json`](schema/change.v1.json) — the vendorable
  `change.json` manifest (the `status`/`tier` enums are the only ESL-owned
  schema).
- **Conformance checker:** [`conformance/esl-conformance.sh`](conformance/esl-conformance.sh).
- **Templates:** [`templates/`](templates/) — per-tier change-folder skeletons.
- **Examples:** [`examples/`](examples/) — one worked change per tier.

## The anti-scope clause (load-bearing)

> A conforming ESL artifact MAY **name** lifecycle fields and **reference** a
> deferred schema, performative, or memory layer **by version**. It MUST NEVER
> **re-declare** an artifact schema, a performative, or a memory layer.
> Re-declaration is a contract bug.

This is what keeps ESL thin. The ONLY ESL-owned schema is the `status` and
`tier` enums in `schema/change.v1.json`. Everything else is borrowed by
reference.

## OWNS / DEFERS

| ESL OWNS | ESL DEFERS (referenced by version, never re-declared) |
|---|---|
| Lifecycle `status` vocabulary + transitions (the state machine) | Artifact schema → **SPECTRA** `spec.{md,yaml}` |
| `change_id` / supersede / diff grammar | Transport → **ECL v1.0** envelopes + SHA-256 verify gate |
| The right-sizing gate (size → route classifier) | Persistence → **CRYSTALIUM** bi-temporal `update` + Semantic/Execution layers |
| The `drift_check` transition + spec-of-record rule | Bus → **Junction** `plan_dispatch` |

## The lifecycle in one line

```
proposed → [deliberated] → in_progress → verified → archived
```

A mandatory mechanical **right-sizing gate** at `proposed` entry classifies each
change on three observable signals (files-touched estimate, SPECTRA /12 score,
trade-off-present boolean) into one of three tiers:

- **trivial** — Kupo micro-fix, **no spec**; bypasses the state machine.
- **lite** — a SPECTRA one-page spec; path `0→2→3→4` (deliberation skipped).
- **full** — the full lifecycle `0→1→2→3→4`.

Promotion to `verified` requires a verify envelope whose author identity is
**distinct from the maker** (maker ≠ checker). Before `archived`, a
`drift_check` re-derives the change's acceptance against the durable
spec-of-record. See [`spec/esl-1.0.md`](spec/esl-1.0.md) for the normative
detail.

## Where a change lives

In a consumer project, a change is a per-change folder under SPECTRA's existing
`.spectra/` root (ESL adds `changes/` alongside `plans/`/`state/`/`logs/`):

```
.spectra/changes/<change_id>/
├── change.json                 # the ESL manifest (schema/change.v1.json)
├── spec.md                     # tier-dependent (lite/full) — SPECTRA-owned
├── spec.yaml                   # tier-dependent (full) — SPECTRA-owned
├── *.envelope.json             # ECL sidecars (propose/critique/verify)
└── archive/<date>-<change_id>/ # on archive: snapshot of the change folder
```

## Quick start — check a change folder

```bash
git clone https://github.com/Rynaro/eidolons-esl /tmp/esl

# Warn mode (default — report, exit 0):
bash /tmp/esl/conformance/esl-conformance.sh .spectra/changes/my-change/

# Block mode (exit 3 on a hard violation — use in CI / verify gates):
bash /tmp/esl/conformance/esl-conformance.sh .spectra/changes/my-change/ --mode block
```

Exit codes: `0` conformant (or warnings-only in `--mode warn`); `1` usage error;
`3` a hard violation in `--mode block`. See
[`conformance/README.md`](conformance/README.md).

## Docs

Comprehensive guidance for adopters and contributors:

- **[`docs/rationale.md`](docs/rationale.md)** — Why ESL exists: the research evidence (T1/T2 CONFIRMED failure modes, +1.7% SWE-bench gain from SDD workflow), the three net-new mechanisms (state machine + right-sizing gate + drift-check), and why we chose to reuse Eidolons instead of building a second framework.
- **[`docs/relationship-to-ecl-eiis.md`](docs/relationship-to-ecl-eiis.md)** — How ESL composes with sibling contracts (ECL = wire format, EIIS = install contract, ESL = lifecycle grammar). The three composition points and versioning independence model.
- **[`docs/adoption.md`](docs/adoption.md)** — How to use ESL in a consumer project: the per-change folder layout, the right-sizing gate workflow, tier-specific lifecycle paths (trivial/lite/full), conformance checking, maker/checker enforcement, and practical scenarios.
- **[`docs/roadmap.md`](docs/roadmap.md)** — The next wave (v1.1+): optional `eidolons spec` CLI verb, per-Eidolon adoptions, native baseline eval (GAP-B closure), EARS helpers, Junction dispatch demo, and rejected candidates with rationales.

## What this repo is NOT

- It is **not** an Eidolon. It is not installed into consumer projects.
- It does **not** define a runtime engine — the host LLM is the runtime.
- It does **not** ship an `eidolons spec` CLI verb in v1.0 (the grammar is
  already driven by SPECTRA + `eidolons run --verify` + CRYSTALIUM).
- It does **not** re-declare SPECTRA's spec schema, ECL's performatives, or
  CRYSTALIUM's layers. It references them by version.
- It does **not** publish to npm/pip/brew. Distribution is `git clone`.

## Relationship to ECL / EIIS

```
EIIS  (Rynaro/eidolons-eiis)  — the install contract.
ECL   (Rynaro/eidolons-ecl)   — the wire-format / hand-off contract.
ESL   (this repo)             — the spec-lifecycle contract.
                                 Composes with ECL + EIIS; replaces neither.
```

See [`docs/relationship-to-ecl-eiis.md`](docs/relationship-to-ecl-eiis.md) for the deep dive on composition, versioning independence, and drift tracking.

## Roadmap

See [`docs/roadmap.md`](docs/roadmap.md) for v1.1+ candidates (optional `eidolons spec` CLI verb, per-Eidolon adoptions, native baseline eval, EARS helpers, Junction demo) and rejected alternatives with rationales.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Open an issue first, then a PR against
`main`. A spec change requires a SemVer bump and a new `spec/esl-X.Y.md` file.

## License

MIT. See [LICENSE](LICENSE).
