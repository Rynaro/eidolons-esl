# Templates

Per-tier change-folder skeletons. Copy one into
`.spectra/changes/<change_id>/` and fill the placeholders; the result passes
`conformance/esl-conformance.sh --mode block` for that tier.

| Tier | Files | When |
|---|---|---|
| `trivial/` | `change.json` only (`spec_ref: null`) | files ≤ 2, /12 ≤ 4, no new behavior — Kupo micro-fix, no spec |
| `lite/` | `change.json` + one-page `spec.md` | bounded scope, /12 5–6, single behavior, no trade-off |
| `full/` | `change.json` + `spec.{md,yaml}` | /12 ≥ 7, or a trade-off, or system-wide blast radius |

The `lite/spec.md` and `full/spec.{md,yaml}` stubs carry an anti-scope header:
**SPECTRA owns those artifacts' schema** (see SPECTRA `spec-profile.v1.json`).
The stubs are slots, not schemas — replace their bodies with SPECTRA's real
output. ESL only requires their presence per tier (ESL v1.0 §4, §8).
