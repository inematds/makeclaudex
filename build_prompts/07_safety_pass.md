# Prompt 07 — Safety pass

> Last prompt. Stress-test every error path and make sure the user is never trapped.

---

Read SPEC.md's "Six safety primitives" section. The hook should already have most of these. This pass verifies them and adds the missing ones.

## Audit checklist

Walk every code path and confirm:

1. **ERR trap.** First line of `stop-hook.sh` after the helper definitions installs a trap that returns approve on any error and exits 0. Confirm no path that mutates state runs before the trap.

2. **Atomic writes.** Every state mutation goes through `claudex_state_write` or `claudex_state_set_field` (which use `tmp + rename`). No direct `>` redirects to state files anywhere. Grep the codebase to confirm.

3. **CAS phase transitions.** Every `phase` field write uses `claudex_phase_transition`, NOT `claudex_state_set_field "phase"`. Exception: deliberate "force" sites like `cancel-loop.sh` that override regardless of current phase. Document which sites are which in code comments.

4. **Lockfile + PID liveness.** `claudex_lock_is_active` does `kill -0`. The lockfile is informational only in this architecture — it doesn't gate concurrency (phase does). Don't load-bear the lock for anything important.

5. **Stale loop sweeper.** Runs on every `start-loop.sh` invocation BEFORE the concurrency check. Old completed loops kept on disk for audit are not swept; only state files older than `CLAUDEX_STALE_MINUTES`.

6. **cwd validation.** Hook reads `repo_root` from state and fails open if `pwd != repo_root`.

## New tests

`tests/synthetic-e2e.sh`. Real Codex calls. Spins up a throwaway repo at `/tmp/claudex_synthetic`, runs through one full plan-mode round end-to-end with an actual `codex exec` invocation. Asserts:

- start-loop produces output and creates a valid review_id.
- The hook BLOCKs with run-runner instructions when PLAN.md is present.
- The runner script has the quoted PROMPTEOF heredoc (prevents shell expansion of the prompt content).
- Running the runner exits 0, produces non-empty output, doesn't hit `stdin is not a terminal` errors, doesn't hit auth errors, mentions the topic.
- mark-done.sh + final hook fire produces approve and cleans up the lockfile.
- A subsequent start-loop accepts a new loop (back-to-back works after a completed loop).

This test costs a few cents in Codex tokens. Document that in the README.

## README and SAFETY docs

Write `README.md` at the repo root and `plugins/claudex/docs/SAFETY.md`. The README leads with what the loop actually does in plain English (what you get, who this is for, terminal example). SAFETY.md is the explicit list of what claudex does and does NOT do, files it touches, and the cost expectation (`~25-30k tokens per round, ~75-90k for the default 3 rounds`).

Also `docs/ARCHITECTURE.md` (loop lifecycle deep dive) and `docs/V2_DESIGN.md` (the auto-apply-with-branch-isolation design that's deferred to v2).

## Final test sweep

Run all three test scripts in order:

```bash
bash plugins/claudex/tests/platform-validation.sh
bash plugins/claudex/tests/smoke-test.sh
bash plugins/claudex/tests/synthetic-e2e.sh   # only if a Codex login is configured
```

If platform + smoke pass, the plugin is shippable. Synthetic e2e confirms real Codex actually integrates, but it's not required for unit-level sign-off.

Plus a `bash plugins/claudex/scripts/doctor.sh` from inside a fresh empty repo to verify the install doesn't depend on any leftover state from your dev machine.

Done. The plugin is shippable.
