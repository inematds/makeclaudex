# claudex — single-page spec

## What it is

A Claude Code plugin that drives an autonomous loop: Claude drafts `PLAN.md`, Codex pressure-tests it, Claude revises, repeat until both agree. Then a closing summary lands in chat. Plus a sibling review mode that audits a diff.

## Hard guarantees

- **Fail open everywhere.** Any unhandled error in the hook returns `{"decision":"approve"}` so the user never gets trapped in a broken loop.
- **No code edits outside its own state directory and the marquee artifacts** (`PLAN.md` for plan mode, `reviews/` for review mode).
- **Phase-based concurrency check.** Two loops at once is refused with a clear message.
- **Atomic state writes.** Every state mutation goes through `tmp + rename`.
- **Stale loops auto-swept** after 15 minutes (configurable via `CLAUDEX_STALE_MINUTES`).

## Lifecycle (plan mode)

```
drafting → reviewing → summarizing → done
                ↑           ↓
                └─ revise ──┘
```

| Phase | Who acts | Hook does |
|---|---|---|
| `drafting` | Claude writes `PLAN.md` | Verifies file exists, CAS to `reviewing`, writes runner, BLOCK with run-runner instructions |
| `reviewing` | Claude runs the runner (Codex), reads findings, either revises PLAN.md OR calls `mark-done.sh` | If `decision_signal=no-material-findings` → CAS to `summarizing`, BLOCK with summary. Else if round + 1 > max → CAS to `summarizing` with `signal=max-reached`, BLOCK with max-rounds summary. Else → increment round, write next-round runner, BLOCK |
| `summarizing` | Claude prints the summary BLOCK content to the user, ends turn | CAS to `done`, cleanup runner/prompt/lock, approve |
| `done` | Terminal | Cleanup if anything left, approve |
| `cancelled` / `errored` | Terminal | Approve |

## Lifecycle (review mode)

```
reviewing → done
```

Single-shot in v1. Hook writes a runner that asks Codex to produce findings + proposed-fixes files in `reviews/`. Read-only. v2 will add `applying` phase with branch isolation.

## State file

`.claude/claudex/<review_id>.state` — YAML-ish key/value, one per line. Per-loop file (NOT a single shared file).

| Field | Type | Notes |
|---|---|---|
| `mode` | `plan` \| `review` | |
| `phase` | `drafting` \| `reviewing` \| `summarizing` \| `done` \| `cancelled` \| `errored` | |
| `topic` | quoted string | user-supplied; multi-line collapsed to single line at write time |
| `round` | int | 1..max_rounds |
| `max_rounds` | int | default 3 (env: `CLAUDEX_MAX_PLAN_ROUNDS`); `--rounds N` overrides |
| `from_draft` | bool | plan mode only |
| `interview_used` | bool | plan mode only; recorded by `--interviewed` flag |
| `review_id` | string | `^[0-9]{8}-[0-9]{6}-[0-9a-f]{6}$` |
| `repo_root` | absolute path | written at start; hook fails open if cwd diverges |
| `session_id` | string | `$CLAUDE_SESSION_ID` |
| `started_at` | ISO 8601 UTC | |
| `started_at_epoch` | int | for elapsed calc |
| `last_updated_at` | ISO 8601 UTC | refreshed on every mutation |
| `decision_signal` | `none` \| `no-material-findings` \| `max-reached` | |

## Sidecar files (per loop)

```
.claude/claudex/<review_id>.lock       PID of the start-loop script (liveness debug only)
.claude/claudex/<review_id>-runner.sh  Generated bash script Claude invokes to call Codex
.claude/claudex/<review_id>-prompt.txt Prompt passed to codex exec via stdin
.claude/claudex/<review_id>/findings-round-N.md   Codex's clean per-round bullet summary
```

## Communication channels

| From | To | Channel |
|---|---|---|
| Hook | Claude | stdout JSON: `{"decision":"approve"}` or `{"decision":"block","reason":"..."}` |
| Claude | Codex | runner script with prompt heredoc'd to a file; `codex exec --dangerously-bypass-approvals-and-sandbox < prompt.txt` |
| Codex | Claude | runner stdout (full transcript) AND `findings-round-N.md` (clean bullets) |
| Claude | Hook | state file mutations (`mark-done.sh` sets `decision_signal=no-material-findings`); `phase` stays `reviewing` so the hook drives the summary path |

## Personas (plan mode, per round)

- **Round 1 — senior engineer.** Design flaws, broken assumptions, ambiguous specs.
- **Round 2 — security and data-integrity reviewer.** Auth, validation, race conditions, partial-failure recovery, secrets, audit, data loss.
- **Round 3+ — ops and SRE reviewer.** Rollback safety, observability, gradual rollout, version skew, on-call ergonomics. Re-runs deepen the prior angles rather than going generic.

Persona stanzas live in `scripts/personas.sh` and are prepended to the Codex prompt by the hook each round.

## Six safety primitives

1. **ERR trap fail-open.** First thing the hook does is install `trap '... approve; exit 0' ERR`.
2. **Atomic state writes.** Every write goes through `tmp + rename` (atomic on the same filesystem).
3. **CAS phase transitions.** `claudex_phase_transition` only writes if current phase matches the expected `from`. Prevents two paths racing to advance the same loop.
4. **Lockfile + PID liveness.** Stored PID can be tested with `kill -0`.
5. **Stale loop sweeper.** `find -mmin +15` on every `start-loop.sh` invocation removes abandoned loops.
6. **cwd validation.** State stores `repo_root`; hook fails open if `pwd != repo_root`.

## Environment variables

| Variable | Default |
|---|---|
| `CLAUDEX_MAX_PLAN_ROUNDS` | 3 |
| `CLAUDEX_MAX_REVIEW_ROUNDS` | 3 |
| `CLAUDEX_STALE_MINUTES` | 15 |
| `CLAUDEX_STATE_DIR` | `.claude/claudex` |

## File tree

```
claudex/
├── .claude-plugin/marketplace.json
└── plugins/claudex/
    ├── .claude-plugin/plugin.json
    ├── commands/
    │   ├── plan.md
    │   ├── review.md
    │   ├── status.md
    │   ├── doctor.md
    │   ├── cancel.md
    │   └── rollback.md
    ├── hooks/
    │   ├── hooks.json
    │   └── stop-hook.sh
    ├── scripts/
    │   ├── start-loop.sh
    │   ├── mark-done.sh
    │   ├── status.sh
    │   ├── doctor.sh
    │   ├── cancel-loop.sh
    │   ├── rollback-loop.sh
    │   ├── state-helpers.sh
    │   ├── personas.sh
    │   └── prompts/
    │       ├── plan-mode-init.md
    │       ├── plan-mode-from-draft.md
    │       └── review-mode-init.md
    └── tests/
        ├── platform-validation.sh
        ├── smoke-test.sh
        └── synthetic-e2e.sh
```
