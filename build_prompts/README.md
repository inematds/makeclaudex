# Build claudex yourself, prompt by prompt

This folder is the "if you wanted to build claudex from scratch in Claude Code, here's the actual sequence of prompts you'd send" walkthrough. Each numbered file is one prompt. Run them in order.

The point isn't to copy-paste them word for word. The point is to see *the order* a feature like this gets built in: skeleton first, state machine before lifecycle, lifecycle before personas, personas before polish.

You'll also want `SPEC.md` open in another tab. It's a single-page architecture reference Claude can consult any time.

## The seven prompts

| # | File | What it builds |
|---|---|---|
| 1 | [`01_scaffold.md`](01_scaffold.md) | Plugin skeleton, slash commands, `hooks.json`, fail-open Stop hook |
| 2 | [`02_state_machine.md`](02_state_machine.md) | Per-loop state files, atomic writes, CAS phase transitions, helpers |
| 3 | [`03_stop_hook_lifecycle.md`](03_stop_hook_lifecycle.md) | The hook brain. Draft → review → done. Runner-script generation. Real Codex call. |
| 4 | [`04_findings_and_personas.md`](04_findings_and_personas.md) | Findings-file extraction for token economy. Per-round persona rotation. |
| 5 | [`05_summary_phase.md`](05_summary_phase.md) | Summarizing phase so the loop lands visibly instead of stopping silently. |
| 6 | [`06_utility_commands.md`](06_utility_commands.md) | `/claudex:status`, `/claudex:doctor`, `/claudex:cancel`, `/claudex:rollback`. |
| 7 | [`07_safety_pass.md`](07_safety_pass.md) | Make every error path fail open. ERR trap, lockfile, sweeper, cwd validation. |

## How to use

1. Open Claude Code in an empty folder.
2. Drop `SPEC.md` into the folder so Claude can read it.
3. Paste the contents of `01_scaffold.md` and let Claude scaffold.
4. Test what it built. Confirm the plugin loads in Claude Code.
5. Move to `02_state_machine.md`. Repeat.
6. Each prompt assumes the previous step is complete and tested.

## Why this works

Most "build it for me" Claude sessions fail because the user dumps a 500-line spec into one prompt and Claude tries to write everything at once. Every layer that goes on top of an unverified layer is a place a bug hides.

Claudex was built incrementally because the systems guarantees demand it. State files have to actually round-trip before you wire the hook to read them. The hook has to fail open before you let it write user-visible BLOCK reasons. Personas only matter after the lifecycle works. Each prompt here represents one "verify before continuing" boundary.
