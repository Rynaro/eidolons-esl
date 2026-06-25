# Conformance fixtures

Genuinely-violating change folders that prove the checker blocks each MUST in
`--mode block`. Each is a real violation (proven RED), not a vacuous same-input
re-run.

| Fixture | Check | `--mode block` exit | Why |
|---|---|---|---|
| `maker-equals-checker/` | C4 | 3 | status=verified, maker==checker, and verify envelope `from.eidolon`==maker — self-verify is blocked |
| `archive-no-drift/` | C5 | 3 | status=archived with `drift_checked:false` — drift_check must run before archive |
| `full-missing-spec/` | C3 | 3 | tier=full with no `spec.{md,yaml}` present |
| `lite-missing-spec/` | C3 | 3 | tier=lite with no one-page `spec.md` present |
| `trivial-no-spec/` | C3 | 0 | tier=trivial with no spec — NOT a violation (the trivial bypass) |
| `ears-complete/` | C7 | 0 | EARS-complete item (all 4 fields) → C7 `ok`; advisory, exit 0 |
| `ears-missing-field/` | C7 | 0 | EARS item missing `then` → C7 `fail` in `--json`, YET exit 0 in block mode (C7 is SHOULD-level; only C1–C6 block) |

`ears-missing-field/` is the load-bearing advisory proof: a genuine C7 `fail`
that does NOT escalate the exit code. The conformant tier examples under
`../../examples/` (including the EARS-form `lite-ears-complete/`) exit 0 in block
mode (positive controls).
