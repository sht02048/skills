---
name: orca-orchestration
description: Coordinate one or more tickets from isolated Orca worker worktrees through implementation, validation, pull request, merge judgment, and clean teardown of the finished worktree. Use when the user asks to process tickets or a feature with Orca orchestration, to implement tickets in parallel across worktrees, to keep the coordinator code-free, or to merge only after React Doctor and the automatic Codex PR review pass; or says "오르카로 처리", "Orca 워크트리", "티켓 병렬로 구현". Typical entry point is the last step of the grill-me → to-spec → to-tickets → implement-spec workflow, when tickets exist and need delegated implementation, but any ticket-shaped work qualifies. Do not use it just to run a one-off Orca worker and get a result back; the installed `orchestration` skill covers that.
---

# Orca Orchestration

Run the delivery through Orca's tracked worktree and orchestration state. The
coordinator directs workers and decides gates; it never edits code.

The usual entry point is the end of the grill-me → to-spec → to-tickets →
implement-spec workflow: the spec is settled, the tickets exist with their
blocking edges, and each ticket needs an isolated implementation. This skill
is not limited to that workflow; any ticket-shaped requirement with a clear
source and acceptance criteria fits.

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

### 1. Start the Workers

Start one worker per ticket with a self-contained task spec (see step 2).
Independent tickets get a new top-level workspace from the repository default
base and may run in parallel. When tickets declare blocking edges (as
to-tickets output does), respect them: start a blocked ticket only after the
tickets it depends on are merged, or stack it on the parent branch when the
user asked for that. Then end the turn and let Orca wake the coordinator, as
the guide requires.

Parallel workers share nothing but the repository. Keep their file ownership
disjoint, and when two tickets must touch the same file, serialize them.

### 2. Write the Task Spec

The worker's spec must include:

- the exact requirement or ticket source;
- repository instructions and scope boundaries;
- worker ownership of every code change;
- required tests and observable QA;
- running React Doctor on the final head SHA;
- commit, push, and PR creation;
- cleanup of the worker's own temporary state before it reports: scratchpad
  repository copies, build output, and any dev server or daemon it started,
  keeping only reports and tools another ticket needs (see step 6 for why);
- a final report containing the PR URL, exact head SHA, checks run, the React
  Doctor command and result for that SHA, what was cleaned up and what was
  intentionally kept for hand-off, and unresolved risks.

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

Merge with `gh pr merge <number> --squash --delete-branch`. After merging,
verify the PR state and merge commit, then tear the workspace down (step 6).

### 6. Hand Off, Then Leave Nothing Behind

A finished ticket must not leave its workspace, caches, build output, or
processes on the machine. This rule exists because of a real incident: during
a parallel refactor, each worker kept repository copies and `dist-*` output in
its scratchpad, and sixteen merged worker worktrees were never removed. The
disk filled up (ENOSPC), the shell itself could no longer start, and every
running agent stopped at once. Treat teardown as part of delivery, not an
afterthought.

Order matters: hand off first, then clean.

1. **Hand off what the next step needs.** Before deleting anything, make sure
   the final report, unresolved risks, and any artifact another ticket or the
   user still needs (measurements, comparison tools, screenshots, notes) are
   recorded somewhere outside the workspace: the PR, a ticket comment, the
   coordinator's report, or a location the user named. If a later ticket
   depends on files from this one, pass them along explicitly rather than
   leaving them in the dying worktree to be found.
2. **Stop the worker's processes.** Dev servers, daemons, preview servers, and
   watchers started by the worker outlive its turn. Close every terminal in
   the workspace through Orca (the `orca-cli` guide covers `terminal close
   --worktree ... --all`) and confirm the close was verified. If Orca reports
   the close as unverifiable, name the process to the user instead of
   claiming it stopped.
3. **Remove the worker's temporary copies.** The worker's task spec must
   already require it to delete its own scratchpad repository copies and
   build output when it finishes, keeping only reports and shared tools. If
   the worker's final report does not confirm this, send a cleanup follow-up
   before removing the worktree.
4. **Remove the worktree.** Once the PR is merged and the hand-off is done,
   remove the Orca worktree (the `orca-cli` guide covers `worktree rm`) and
   let the merged branch go. Do this right after recording the merge, not at
   the end of the whole batch; the accumulation across many tickets is what
   fills the disk.
5. **Verify the result.** Confirm the worktree no longer appears in Orca's
   worktree list and that free disk space did not shrink across the ticket.
   Report what was removed and anything you could not remove.

Keep the worktree only when the user says so, or when merge is withheld and
the fix is still pending. A withheld PR keeps its workspace; a merged one
never does.

Also cap concurrency when workers run heavy comparisons or builds: more than
two workers running end-to-end comparisons at once was the trigger in the
incident above. Stagger them or tell the user when the ticket set calls for
more.

Report per ticket: the workspace and whether it was removed, PR, merged or
withheld decision, exact SHA, React Doctor result, Codex review signal, what
was handed off, and any remaining risk.
