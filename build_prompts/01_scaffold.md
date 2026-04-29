# Prompt 01 — Scaffold

> Paste this into Claude Code in an empty folder. Have `SPEC.md` open in another tab so Claude can consult it.

---

I want to build a Claude Code plugin called `claudex`. Read `SPEC.md` if it's in this folder; that's the source of truth.

For this first pass, just scaffold the layout. No real logic yet. I want to verify the plugin loads in Claude Code before we wire up anything substantive.

Build:

1. The marketplace manifest at `.claude-plugin/marketplace.json` and the plugin manifest at `plugins/claudex/.claude-plugin/plugin.json`. Author name `promptadvisers`. Description matches the spec.
2. Six slash command files under `plugins/claudex/commands/`: `plan.md`, `review.md`, `status.md`, `doctor.md`, `cancel.md`, `rollback.md`. Each one for now is a placeholder that just `echo`s "TODO" so I can confirm the commands surface in Claude Code's slash-command list. Use proper YAML frontmatter (`description`, `allowed-tools: Bash`).
3. `plugins/claudex/hooks/hooks.json` registering a `Stop` hook that calls `${CLAUDE_PLUGIN_ROOT}/hooks/stop-hook.sh` with a 60s timeout and a `statusMessage`.
4. `plugins/claudex/hooks/stop-hook.sh`. **Critical**: this is fail-open everywhere from day one. Install an `ERR` trap that prints `{"decision":"approve"}` on any unhandled error and exits 0. The default body for now: read stdin, log "no logic yet", print approve. The user must NEVER get trapped because the hook crashed.
5. `plugins/claudex/tests/platform-validation.sh`. A simple bash test that asserts the hook is executable, returns valid JSON when fed `{}` on stdin, and fail-opens on garbage input. Print colored ✓/✗ per check. Exit non-zero on any failure.

Don't write any state-machine code yet. That's the next prompt. Just the skeleton + the unconditional fail-open hook.

Once you're done, run `bash plugins/claudex/tests/platform-validation.sh` and tell me what it says.
