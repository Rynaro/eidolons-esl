# Contributing to ESL

ESL (Eidolons Spec Lifecycle) is a small standards repo. Contributions fall into
four categories; the bar and process differ for each.

## 1. Spec changes (`spec/esl-X.Y.md`)

These are normative. They affect every Eidolon that follows the lifecycle.

- **Open an issue first** describing the intent and the SemVer impact.
- An additive change (a new OPTIONAL field, a new SHOULD) goes in the next minor.
  Create a new file `spec/esl-1.x+1.md` rather than editing the previous one.
- A change that removes a field or tightens a SHOULD into a MUST is a **major**
  (`v2.0`).
- **Anti-scope is P0.** A spec change MUST NOT re-declare a SPECTRA artifact
  schema, an ECL performative as ESL-owned, or a CRYSTALIUM memory layer. Name
  and reference them by version. The only ESL-owned schema is the `status` /
  `tier` enums.
- Keep `spec/esl-X.Y.md` ≤ ~300 lines and hand-written. A contract that preaches
  anti-bloat must not ship bloated.

## 2. Schema changes (`schema/change.v1.json`)

- JSON Schema **2020-12**. Run `jq empty schema/change.v1.json` locally.
- Adding an OPTIONAL field is additive (next minor). Removing or tightening a
  field is a major.
- The `maker != checker` inequality is intentionally NOT in the schema (JSON
  Schema cannot express field-inequality portably) — it is a conformance-checker
  rule. Do not try to encode it in the schema.

## 3. Conformance changes (`conformance/`)

- **Bash 3.2 only** — no `declare -A`, no `${var,,}`/`${var^^}`, no
  `readarray`/`mapfile`, no `&>>`. macOS ships bash 3.2 as the system shell.
- Run `shellcheck -S error conformance/esl-conformance.sh`.
- Keep it deterministic and offline — no `curl`/`wget`/`nc`; identical input
  yields byte-identical `--json` stdout.
- Add a fixture under `conformance/tests/` for every new failure mode. Prove it
  RED before the fix (a same-input re-run that passes vacuously is not a test).
- Exit codes are normative — see §8 of the spec (`0`/`1`/`3`; `2` reserved).

## 4. Templates / examples (`templates/`, `examples/`)

- Copying a template + filling placeholders MUST produce a change folder that
  passes `conformance/esl-conformance.sh --mode block` for that tier.
- The `lite`/`full` spec stubs MUST carry the anti-scope header. Examples must
  keep maker ≠ checker on their verify envelopes.

## Pull requests

- One concept per PR. Title: `<area>: <imperative description>`.
- Link the issue; CI MUST be green before review.

## Releasing

- `ESL_VERSION` MUST match the `MAJOR.MINOR` of `spec/esl-X.Y.md`.
- Update `CHANGELOG.md` `[Unreleased]` → `[X.Y.Z] — YYYY-MM-DD` before tagging.

## License

By contributing you agree your contribution is licensed under MIT.
