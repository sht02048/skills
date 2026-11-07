---
name: orca-orchestration
description: Coordinate a requirement or ticket from an isolated Orca worktree through implementation, validation, pull request, and merge judgment. Use when the user asks to process a ticket or feature with Orca orchestration, create an Orca worktree for delegated implementation, keep the coordinator code-free, or merge only after React Doctor and the automatic Codex PR review pass, or says "오르카로 처리" or "Orca 워크트리".
---

# Orca Orchestration

Run the delivery through Orca's tracked worktree and orchestration state. The
coordinator directs workers and decides gates; it never edits code.

This skill defines the delivery policy only. How to drive Orca (starting
workers, messaging, being woken, recovery) comes from the installed
`orchestration` skill, which loads a guide matched to the Orca binary in use.

## Load the Orca Guide First

1. Load the installed `orchestration` skill and follow its instructions to
   resolve the Orca executable for this session.
2. Run `ORCA skills get orchestration` (with the resolved executable) and read
   the guide before issuing any Orca command. Fetch it once per session.
3. Follow the guide for every Orca mechanic: starting a worker, choosing its
   workspace, sending and replying to messages, handling wake-ups, and
   recovering a failed or stalled worker. Never reconstruct Orca commands from
   memory or from this file.
4. If the installed skill is missing or `skills get` fails, stop and report
   the exact error to the user. Do not fall back to a non-Orca subagent tool.

Where this file and the Orca guide disagree on how to operate Orca, the Orca
guide wins. Where they disagree on what to deliver and when to merge, this
file wins.

## Non-Negotiable Ownership

- Give every requirement or ticket its own Orca worker workspace. Let the
  worker start create it, as the Orca guide describes; do not reuse the
  coordinator checkout for worker edits.
- Only tracked Orca workers count as orchestration. A terminal prompt without
  an Orca Task and Dispatch is not orchestration, and must never be described
  as such.
- Assign all repository mutations to workers: code, tests, generated files,
  formatting, commits, pushes, PR creation, and review fixes.
- Never let the coordinator run an editor, `apply_patch`, a formatter that
  writes files, `git commit`, or any equivalent code-changing command. The
  coordinator may inspect state, issue Orca/GitHub control commands, make
  decisions, and merge.
- Keep one explicit owner per file or responsibility when multiple workers run.
  Shared workspaces make overlapping edits unsafe.

## Delivery Flow

### 1. Start the Worker

Start one worker per independent ticket with a self-contained task spec (see
step 2). Independent tickets get a new top-level workspace from the repository
default base. Stack a ticket on a parent or non-default base only when the
user asked for that dependency. Then end the turn and let Orca wake the
coordinator, as the guide requires.

### 2. Write the Task Spec

The worker's spec must include:

- the exact requirement or ticket source;
- repository instructions and scope boundaries;
- worker ownership of every code change;
- required tests and observable QA;
- running React Doctor on the final head SHA;
- commit, push, and PR creation;
- a final report containing the PR URL, exact head SHA, checks run, the React
  Doctor command and result for that SHA, and unresolved risks.

### 3. Supervise Without Editing

- Act on each message Orca wakes the coordinator with. Answer worker
  questions with evidence and judgment; ask the user when the answer does not
  follow from the ticket or their instructions.
- On a defect or failed gate, send the correction to a worker. The coordinator
  must not patch it.
- Silence or a timeout alone never proves the worker exited. Use the Orca
  guide's liveness and recovery steps. If the worker appears stalled and the
  guide's steps do not restore progress, report the stalled state to the user.
- Stop immediately when the user says to stop.

### 4. Evaluate the Pull Request

Bind every decision to the PR's current head SHA. A pass for an older SHA is
stale after any new commit.

Minimum merge gates:

1. The worker's report shows React Doctor completed successfully for the
   current head SHA. A result for another SHA, or no reported command, is not a
   pass; send a rerun to the worker.
2. The automatic Codex PR review completed and passed for the current head SHA.

Interpret the automatic Codex review on the PR:

```text
👀 on the automatic review item → review is still running; wait
👍 → review completed with no blocking finding
Neither 👀 nor 👍 on the automatic review item → Codex posted a separate review; fetch and read it before deciding
```

- A separate Codex review is a pass only when it reports no blocking issue for
  the current head SHA. The absence of reactions alone is never a pass. If the
  review contains findings, send fixes to a worker and wait for the automatic
  review of the new SHA.
- Never post `@codex review`, call a review-triggering API, or otherwise start
  Codex review manually. Opening or updating the PR starts the configured
  automatic review.
- If no automatic completion signal appears, do not manufacture one and do not
  merge. Report the pending gate.

React Doctor and Codex are the minimum floor, not an instruction to ignore
failed required CI, unresolved review findings, merge conflicts, unmet
requirements, or user-requested validation. A material risk is any security,
authorization, data-loss, or money-handling concern found in the diff or
reviews.

### 5. Decide Whether to Merge

Use this decision tree:

```text
Is the PR head SHA unchanged since validation?
├── No → invalidate prior gates and wait for current-SHA results
└── Yes
    ├── React Doctor passed?
    │   └── No → do not merge
    ├── Automatic Codex review passed?
    │   └── No or still 👀 → do not merge
    ├── Any required CI failure, unresolved finding, conflict, unmet requirement, or material risk?
    │   └── Yes → do not merge; send a fix to a worker or report the blocker
    └── No → the coordinator may merge
```

Passing the minimum gates permits merge consideration; it never forces a merge.

Merge with `gh pr merge <number> --squash --delete-branch`. Leave the Orca
worker workspace in place.

After merging, verify the PR state and merge commit. Report the workspace, PR,
merged or withheld decision, exact SHA, React Doctor result, Codex review
signal, and any remaining risk.
