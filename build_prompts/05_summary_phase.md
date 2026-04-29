# Prompt 05 — The summarizing phase

> Without this, the loop ends silently. Claude's turn just stops with no closing message and the user is left wondering whether anything happened. This prompt fixes that.

---

Add a new phase between `reviewing` and `done` called `summarizing`. The job of this phase is to BLOCK once with a final summary so Claude prints it to the user, then approve on the next fire.

## Lifecycle change

```
Before:  drafting → reviewing → done
After:   drafting → reviewing → summarizing → done
```

`summarizing` is treated as an **active** phase by `start-loop.sh`'s concurrency check (it's not terminal yet).

## Hook changes (`stop-hook.sh`)

1. Add a `build_rounds_table <final_round>` helper near the top. Iterates `1..final_round`, reads each findings file, calls `claudex_findings_severity_counts`, pairs with the persona label. Returns lines like:

   ```
   - Round 1 (Senior-engineer review): high=2 medium=4 low=1
   - Round 2 (Security and data-integrity review): high=2 medium=2 low=0
   ```

   If a round's findings file is missing, write "no findings file" instead of the counts. Important: bash command substitution strips trailing newlines, so callers need to handle that.

2. In the `reviewing` case, when `decision_signal=no-material-findings`:
   - CAS `reviewing → summarizing` (NOT to `done`).
   - Build the summary using `build_rounds_table` and `format_elapsed`.
   - BLOCK with a markdown message titled `### Claudex plan loop complete ✓`. Body: rounds run, total time, findings table, then "**Print this summary to the user so they see the loop landed.** Then end your turn. The Stop hook will allow exit cleanly."

3. In the `reviewing` case, when round overflow:
   - Set `decision_signal=max-reached`.
   - CAS `reviewing → summarizing`.
   - BLOCK with `### Claudex plan loop stopped at max rounds (round N of M)`. Include the rounds table, the path to the final round's findings file, and three explicit options: revise manually, re-run with `--rounds N` larger, or accept as known-incomplete.

4. Add a `summarizing` case:
   - CAS `summarizing → done`.
   - Cleanup runner, prompt file, lockfile.
   - Log elapsed time.
   - Approve.

## `mark-done.sh` change

`mark-done.sh` currently sets `phase=done` and `decision_signal=no-material-findings`. Change it to **only** set `decision_signal=no-material-findings` and **leave `phase=reviewing`**. The next hook fire will see the signal and drive the summarizing path.

Update its echoed line to say "Loop marked as done. Stop hook will deliver the summary on next fire."

## Format gotcha

Bash double-quoted heredoc strings interpret backticks as command substitution. The summary BLOCK uses triple-backticks for code fencing. Escape every backtick that should render as a literal in the final output: `\``. Test that the JSON output of the BLOCK message parses cleanly with `python3 -c 'import json, sys; json.load(open(...))'`.

Also: the rendered summary needs a blank line before the rounds table. Bash command substitution strips the trailing newline of `build_rounds_table`, so the source needs an explicit blank line after `$ROUNDS_TABLE`.

## Mark-done path in BLOCK

While you're in the BLOCK strings, fix one cosmetic issue: the existing BLOCKs say `bash $CLAUDE_PLUGIN_ROOT/scripts/mark-done.sh $REVIEW_ID`. In real installs this resolves to a noisy `~/.claude/plugins/marketplaces/.../scripts/mark-done.sh` path. Change to literal `bash \${CLAUDE_PLUGIN_ROOT}/scripts/mark-done.sh $REVIEW_ID` so Claude renders the env-var form.

## Tests

Update `tests/smoke-test.sh`:

- After `mark-done.sh`, the next hook fire should BLOCK (not approve) with a summary mentioning "plan loop complete" and "Findings by round". State phase should be `summarizing`.
- The fire AFTER that should approve. State phase should be `done`.
- For the back-to-back-after-completion test (P2 fix), update the "fire hook to clean lockfile" step to be TWO fires now (summarizing + done).
- Add a section that triggers the max-rounds path: start a `--rounds 1` loop, write PLAN.md, fire hook to enter reviewing, fire again to trigger overflow. Assert the BLOCK mentions "stopped at max rounds" and state goes to `summarizing` with `decision_signal=max-reached`. Then fire once more and assert approve.
- Add a section that asserts the BLOCK contains the literal `${CLAUDE_PLUGIN_ROOT}` placeholder, NOT the resolved absolute path.

Run all tests, send counts.
