# Prompt 03 — Stop-hook lifecycle

> The state machine is verified. Now wire the hook to drive it.

---

Replace the fail-open stub in `plugins/claudex/hooks/stop-hook.sh` with the real lifecycle engine. Read `SPEC.md` for the phase table.

Build:

1. **Helpers at the top.** `log()` appends to `.claude/claudex/log` with UTC timestamp. `approve(reason)` logs and prints `{"decision":"approve"}`. `block(reason)` logs the first 80 chars, JSON-escapes the reason via `python3 -c 'import sys, json; print(json.dumps(sys.stdin.read()))'` with a sed-based fallback if python3 is missing, then prints `{"decision":"block","reason":...}`.

2. **ERR trap unconditional.** First active line of the script. Catches anything not explicitly handled and fails open.

3. **Source state-helpers.sh** (or fail-open if it's missing).

4. **Find the active loop.** If none, approve. If the review_id format is invalid, remove the bad state file and approve.

5. **Read state fields.** mode, phase, round, max_rounds, decision_signal, topic, repo_root, started_at_epoch.

6. **cwd validation.** If `repo_root` differs from current `pwd`, fail open with a log line. The hook may have been triggered in a different project than where the loop began.

7. **Validate numerics.** Garbage `round` or `max_rounds` falls back to 1 / max-default.

8. **`write_runner_script` helper.** Builds a self-contained bash script at `.claude/claudex/<id>-runner.sh`. The script writes the prompt to a heredoc with `<<'PROMPTEOF'` (single quotes — critical: prevents shell expansion of the prompt content) then runs `codex exec --dangerously-bypass-approvals-and-sandbox < $PROMPT_FILE`. Print the persona and round number in the runner's header comment + first echo line.

9. **Plan mode case statement.**
   - `drafting`: if `PLAN.md` missing or empty → BLOCK telling Claude to draft it. If present → CAS to `reviewing`, build the round-1 Codex prompt that asks for an adversarial review and writes findings to `.claude/claudex/<id>/findings-round-1.md` in a strict `## High / ## Medium / ## Low` bullet format. Tell Codex to write exactly "No substantive findings." if there's nothing material. Write the runner. BLOCK with a markdown-formatted message instructing Claude to run the runner, read the findings file, and either revise PLAN.md (with a `## Changelog` section) or call `mark-done.sh`.
   - `reviewing`: if `decision_signal=no-material-findings` → loop is complete (we'll add the summarizing phase in prompt 05; for now just CAS to `done`, cleanup, approve). Otherwise increment round; if new round > max_rounds, set `decision_signal=max-reached`, CAS to `done`, cleanup, approve. Otherwise: write the next-round runner, BLOCK with the new instructions.
   - `done`: cleanup any leftover artifacts, approve.
   - any other phase: log unknown phase, approve.

10. **Review mode case statement.**
    - `reviewing`: write a runner that asks Codex to review the current diff and produce `reviews/review-<id>.md` plus `reviews/proposed-fixes-<id>.md`. CAS to `done`. BLOCK with a brief "review starting" message.
    - `done`: cleanup, approve.

11. **`plugins/claudex/scripts/mark-done.sh`.** Takes a review_id arg. Validates the format. Sets `decision_signal=no-material-findings` and `phase=done` (we'll change this in prompt 05). Atomic.

12. **Smoke test** at `plugins/claudex/tests/smoke-test.sh`. Spins up a temp git repo. Runs start-loop.sh. Writes a fake PLAN.md. Fires the hook directly (via `echo '{}' | bash $HOOK`) and asserts it BLOCKs with run-runner instructions. Asserts state transitioned to `reviewing`. Asserts a runner script was written. Asserts firing again increments round. Asserts `mark-done.sh` followed by another hook fire produces an approve. Asserts concurrent-loop refusal. Asserts cancel-loop and rollback-loop work. Same colored ✓/✗ format as platform-validation.

Run both test scripts. Tell me the counts.

**Do NOT actually call Codex in any test.** All tests stub the runner or just check that the runner file exists. Real Codex calls live in a separate `synthetic-e2e.sh` we'll add later.
