# Prompt 06 — Utility commands

> Loop works end-to-end. Now add the four utility commands that make it usable in real life.

---

Build:

## `/claudex:status`

`plugins/claudex/commands/status.md` is a one-liner that runs `bash "${CLAUDE_PLUGIN_ROOT}/scripts/status.sh"`. Read-only.

`plugins/claudex/scripts/status.sh`:

- Find the most recent state file (`claudex_find_active_loop`).
- Print mode, phase (color-coded: yellow for active, green for done, red for cancelled/errored), round (cap displayed at max_rounds — the internal counter overshoots by one before max-rounds termination is detected), topic, started_at, elapsed time computed from `started_at_epoch`, decision_signal, interview_used, from_draft.
- Replace lockfile-aliveness check (it's never useful in claudex's between-turns architecture — start-loop's PID exits immediately) with an "activity" line derived from phase: `active`, `complete`, `cancelled`, `errored`.
- For each round's findings file in `.claude/claudex/<id>/`, print a line like `round 1   high=2 medium=4 low=1` using `claudex_findings_severity_counts`.
- Always exit 0. **Do not install an ERR trap.** `claudex_find_active_loop` legitimately returns non-zero when there are no loops, and an ERR trap would print a scary error on the happy "no loops" path.

## `/claudex:doctor`

`plugins/claudex/commands/doctor.md` is a one-liner that runs `bash "${CLAUDE_PLUGIN_ROOT}/scripts/doctor.sh"`.

`plugins/claudex/scripts/doctor.sh`:

- Sections with colored ✓/✗ checks.
- Shell: bash present, bash 4+ (warn on 3.x).
- Codex CLI: in PATH; if present, also print `codex --version`.
- Optional dependencies: python3 (warn-only).
- State directory: `.claude/claudex` exists and is writable.
- Plugin files: collapse the file-existence check for the 15 expected plugin files into a single `all 15 plugin files present` line on success. Itemize only on failure.
- Hook fail-open sanity: hook is executable, returns approve on empty state.
- Helpers smoke: state-helpers + personas source cleanly. Round 1/2/3 personas non-empty and distinct.
- Loop hygiene: print whether there's a loop on disk and its phase.

Exit 1 if any required check fails, 0 otherwise.

Same rule as status: no ERR trap. The check helpers consume exit codes via `if`.

## `/claudex:cancel` and `/claudex:rollback`

Already need scripts. Build:

- `plugins/claudex/scripts/cancel-loop.sh`: find the active loop, set `phase=cancelled`, remove the runner / prompt / lock files. Keep the state file for audit.
- `plugins/claudex/scripts/rollback-loop.sh`: nuke every state file, lock file, runner script, prompt file, and per-review findings folder under `.claude/claudex/`. Keep `.claude/claudex/log` for debugging. Print the count of files removed.

Their respective `commands/cancel.md` and `commands/rollback.md` files are one-liners pointing at the scripts.

## Interview offer in `/claudex:plan`

`plugins/claudex/commands/plan.md` already exists. Extend its procedure:

- If `$ARGUMENTS` is empty, ask the user for a topic, do NOT run start-loop yet.
- If `$ARGUMENTS` does not contain `--skip-interview`, use `AskUserQuestion` with two options: "Interview me first (Recommended)" / "Just launch the loop". On opt-in, ask three more questions (scope, constraints, worry list — provide topic-aware option labels). Compose a single-line enriched topic with `; ` separators (multi-line topics break the YAML state file). Pass with `--interviewed`.
- Otherwise pass `$ARGUMENTS` through verbatim to `start-loop.sh`.

`start-loop.sh` accepts `--interviewed` and writes `interview_used: true` to state. The new `started_at_epoch` and `interview_used` fields go in the initial state too.

## Tests

Update `tests/smoke-test.sh` with sections for:

- `started_at_epoch` and `interview_used` are in the state file after start-loop.
- `--skip-interview` accepted (no-op in start-loop).
- `--interviewed` records `interview_used: true`.
- `/claudex:status` on a fresh loop prints review_id, phase, elapsed.
- `/claudex:status` on empty state directory says "no loops".

Update `tests/platform-validation.sh` to confirm status.sh and doctor.sh both run without crashing on an empty state directory.

Run all tests, send counts. The plugin is now usable end-to-end. Next prompt is the safety pass.
