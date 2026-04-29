# Prompt 02 — State machine

> Run after prompt 01 passes. The hook should currently be a fail-open stub. Now we give it a brain.

---

We need the state machine before we wire the hook lifecycle. Claudex tracks each loop in a YAML-ish file under `.claude/claudex/<review_id>.state`. Per-loop, never shared. See `SPEC.md` for the full field list.

Build:

1. `plugins/claudex/scripts/state-helpers.sh`. Sourceable. Exposes:
   - `claudex_new_review_id` — generates `YYYYMMDD-HHMMSS-XXXXXX` (six lowercase hex chars at the end).
   - `claudex_validate_review_id <id>` — strict regex check; returns 0/1.
   - `claudex_state_write <file> <content>` — atomic. Write to `${file}.tmp.$$` then `mv -f`. Never partial files.
   - `claudex_state_read_field <file> <field>` — grep the YAML-ish value.
   - `claudex_state_set_field <file> <field> <value>` — sed in place via tmp + rename. Also bumps `last_updated_at` to now (UTC ISO 8601). Don't recursively bump itself when you ARE setting `last_updated_at`.
   - `claudex_phase_transition <file> <from_phase> <to_phase>` — compare-and-swap. Only writes if current phase matches `from`. Updates `last_updated_at` in the same write. Returns 0/1.
   - `claudex_lock_write <file>` — writes `$$`.
   - `claudex_lock_is_active <file>` — `kill -0` on the stored PID.
   - `claudex_sweep_stale` — `find -mmin +$CLAUDEX_STALE_MINUTES` (default 15) on `*.state` files; remove the state file plus its `.lock`, `-runner.sh`, `-prompt.txt`, and per-review subfolder.
   - `claudex_find_active_loop` — print path to the most-recent state file (by mtime) or return 1 if none.

2. `plugins/claudex/scripts/start-loop.sh`. Called by `/claudex:plan` and `/claudex:review`. Behavior:
   - Parse flags before the topic: `--rounds N`, `--from-draft`, `--skip-interview`, `--interviewed`. Reject `--rounds 0` and non-integers with a clear error. Reject `--from-draft` if `PLAN.md` is missing.
   - Refuse to start if any existing state file has phase NOT in `{done, cancelled, errored}`. Phase-based concurrency, NOT file-presence. Completed state files are kept on disk for audit.
   - Sweep stale loops first (`claudex_sweep_stale`).
   - Generate a `review_id`, write the initial state file atomically, write a lockfile.
   - Print initial instructions to stdout (the slash command will surface this to Claude). For plan mode, point Claude at `PLAN.md` and tell them to draft it then end their turn. For review mode, tell them to just end their turn (the hook does everything).

3. Extend `plugins/claudex/tests/platform-validation.sh` with sections that exercise every helper: id generation, validation rejecting bad formats including `../../../etc/passwd`, atomic write round-trip, CAS valid + stale, sweeper removing files older than threshold while keeping fresh ones, lockfile alive vs dead PID, active-loop finder returning most-recent.

Hook lifecycle stays a fail-open stub for now. We're just verifying the state machine in isolation.

Run `bash plugins/claudex/tests/platform-validation.sh` and tell me what passes.
