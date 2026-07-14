# macro-routine

Weekly FICC macro analysis, executed as a Claude Code Routine.

- `SPEC.md` — the full analysis specification (authoritative; do not modify)
- `ROUTINE_PROMPT.md` — routine execution instructions (do not modify)
- `reports/` — one report per run, `YYYY-MM-DD_macro.md` (the only writable path)

This file is intentionally minimal: routine runs are stateless and every token here is charged on every run. All operating logic lives in SPEC.md and ROUTINE_PROMPT.md.
