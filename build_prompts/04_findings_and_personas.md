# Prompt 04 — Findings extraction + persona rotation

> The lifecycle works in tests. Now make the loop actually pleasant: extract small findings files so Claude doesn't choke on transcripts, and rotate reviewer personas per round.

---

Two changes, both additive:

## A. Findings file extraction

Field testing showed Claude was burning context reading 30k+ token Codex transcripts every round. Have Codex also write a clean small bullet summary that Claude reads instead.

1. Update the round-N Codex prompt in `stop-hook.sh` to include this paragraph at the bottom:

   > CRITICAL OUTPUT REQUIREMENT: After your full analysis, write a clean summary of just the findings (severity + one-line description + recommendation) to the file `<findings_path>`. Use this exact format: `# Round N findings\n\n## High\n- <description> (<recommendation>)\n- ...\n## Medium\n...\n## Low\n...`. If no material findings, the file should contain exactly `# Round N findings\n\nNo substantive findings.`. Write that file before exiting.

2. The hook constructs the findings path as `.claude/claudex/<review_id>/findings-round-<round>.md`. Pre-create the per-review subfolder.

3. The BLOCK message tells Claude: "When Codex finishes, read the clean findings summary from `<path>`. That file is a short bullet list. Skip the full transcript unless you need extra context."

4. Add `claudex_findings_severity_counts <file>` to `state-helpers.sh`. Returns `high=N medium=N low=N` by counting bullet lines under each `## Severity` header. Return `high=0 medium=0 low=0` if the file is missing.

5. On round 2+, prepend a "**Previous round:** high=N medium=N low=N" line to the BLOCK so Claude (and the viewer watching) sees the trajectory before the next round runs.

## B. Persona rotation

Make each round attack from a different angle.

1. Create `plugins/claudex/scripts/personas.sh`. Sourceable. Two functions:
   - `claudex_persona_for_round <N>` — prints a multi-paragraph stanza describing the reviewer persona for round N.
     - Round 1: skeptical senior engineer. Hunts for design flaws and broken assumptions.
     - Round 2: security and data-integrity reviewer. Auth, validation, race conditions, partial-failure recovery, secrets, audit trails, data loss.
     - Round 3+: ops / SRE reviewer. Rollback safety, observability, gradual rollout, version skew, on-call ergonomics. On round 4+, the stanza should say "deepen the previous angles" rather than restating the same thing.
   - `claudex_persona_label_for_round <N>` — prints a one-line label for use in BLOCK headers ("Senior-engineer review", "Security and data-integrity review", "Ops and SRE review").

2. In `write_runner_script`, source personas.sh and prepend the persona stanza to the Codex prompt. Pass the round number through.

3. Update the BLOCK headers to read: `### Claudex round N of M, <persona-label>`.

## Tests

Update `tests/platform-validation.sh`:

- Source `personas.sh` and assert each round's stanza is non-empty and round 1/2/3 differ.
- Synthesize a findings file with known counts, assert `claudex_findings_severity_counts` returns the right tally.
- Assert missing findings file returns the zeroed counts.

Update `tests/smoke-test.sh`:

- After firing through to round 2 then round 3, assert each runner script's persona keyword (round 2 mentions "security", round 3 mentions "ops" or "SRE").
- Assert the round-2 BLOCK contains a "Previous round" tally line.

Run both test scripts. Send the counts.

**Important:** the persona only changes the Codex prompt. The state machine, hook lifecycle, and BLOCK structure stay identical to prompt 03.
