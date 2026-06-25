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

The three conformant tier examples under `../../examples/` exit 0 in block mode
(positive controls).
