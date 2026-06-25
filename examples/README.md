# Examples

One worked change per right-sizing tier, plus the optional EARS acceptance form.
Each passes `conformance/esl-conformance.sh <example>/ --mode block` with exit 0.

| Example | Tier | Demonstrates |
|---|---|---|
| `trivial-typo-fix/` | trivial | no-spec bypass route; `DELEGATE` → Kupo; maker (kupo) ≠ checker (vigil) |
| `lite-add-flag/` | lite | one-page `spec.md`; lite spine `0→2→3→4`; `PROPOSE` + `INFORM(verify_pass)`; maker (vivi) ≠ checker (kupo-verifier); minimal `{id, verify_method}` acceptance form (no C7) |
| `lite-ears-complete/` | lite | the OPTIONAL **EARS** acceptance form `{id, given, when, then, verify_method}`; the advisory C7 lint passes (`ok`); backward-compatible alternative to `lite-add-flag/`'s minimal form |
| `full-new-subsystem/` | full | full lifecycle `0→1→2→3→4`; `PROPOSE`/`CRITIQUE`/`INFORM`; `drift_checked=true` before archive; archive snapshot + Semantic promotion (documented) |

## Conventions

- Every `*.envelope.json` sidecar carries `PARENT_FILLS_SHA256` /
  `PARENT_FILLS_SIZE` placeholders for the ECL SHA fields. The example author
  (or Vivi at build time) fills them; the ESL conformance checker does NOT
  verify SHAs — it only checks envelope well-formedness + that `performative`
  is in the ECL closed-10 set, so the examples conform as-is.
- The `lite` and `full` examples demonstrate **maker ≠ checker**: the verify
  envelope's `from.eidolon` differs from `change.json.maker`. A self-verified
  change is a contract violation (ESL v1.0 §5).
- Semantic-layer promotion at archive is the consumer Eidolon's
  `mcp__crystalium__*` call, not ESL's. ESL only DEFINES the transition.
