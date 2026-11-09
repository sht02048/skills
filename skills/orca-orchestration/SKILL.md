---
name: orca-orchestration
description: Coordinate one or more tickets from isolated Orca worker worktrees through implementation, validation, pull request, merge judgment, and clean teardown of the finished worktree. Workers open and verify each PR with the `pr` skill, whose verdict for the head SHA (project gates in `.agents/pr-gates.json`, CI, the automatic Codex review) is the merge floor. Use when the user asks to process tickets or a feature with Orca orchestration, to implement tickets in parallel across worktrees, to keep the coordinator code-free, or to merge only after the project's checks pass; or says "오르카로 처리", "Orca 워크트리", "티켓 병렬로 구현". Typical entry point is the last step of the grill-me → to-spec → to-tickets → implement-spec workflow, when tickets exist and need delegated implementation, but any ticket-shaped work qualifies. Do not use it just to run a one-off Orca worker and get a result back; the installed `orchestration` skill covers that.
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

## Resolve the Merge Gates

The `pr` skill owns PR verification: which gates a project uses, how to run
them, how to read CI and the automatic Codex review, and the verdict for a
head SHA. This skill never restates those rules; it consumes the verdict.

1. Load the `pr` skill. If it is missing, stop and report it; do not
   verify PRs from memory.
2. Before starting any worker, resolve the gate set by following the `pr`
   skill's "Merge gates" section: read `.agents/pr-gates.json`, or inspect the
   repository and ask the user. Resolving it once here keeps parallel workers
   from each stopping to ask the same question.
3. Put the resolved gate set in every task spec so workers use it without
   asking.
4. If the user chose to save it, do not write the file yourself; it is a
   repository mutation. Start a dedicated config worker whose task spec
   contains the exact JSON for `.agents/pr-gates.json`, and treat its PR like
   any ticket PR. Apply the gate set to the current run immediately rather
   than waiting for that PR to merge.

## Non-Negotiable Ownership

- Give every requirement or ticket its own Orca worker workspace. Let the
  worker start create it, as the Orca guide describes; do not reuse the
  coordinator checkout for worker edits.
- Only tracked Orca workers count as orchestration. A terminal prompt without
  an Orca Task and Dispatch is not orchestration, and must never be described
  as such.
- Assign all repository mutations to workers: code, tests, generated files,
  formatting, configuration files including `.agents/pr-gates.json`,
  commits, pushes, PR creation, and review fixes.
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
- commit, then push and PR creation with the `pr` skill, given the resolved
  gate set, the confirmed base branch, and which screenshot views to capture
  (or that none are needed), so the worker never stops on a question the
  coordinator already settled; after any later commit (a review fix, for example), rerun the
  `pr` skill so it re-verifies the existing PR for the new head SHA;
- cleanup of the worker's own temporary state before it reports: scratchpad
  repository copies, build output, and any dev server or daemon it started,
  keeping only reports and tools another ticket needs (see step 6 for why);
- a final report containing the `pr` skill's verdict report (PR URL, head
  SHA, verdict, each gate's result), other checks run, what was cleaned up
  and what was intentionally kept for hand-off, and unresolved risks.

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

The merge floor is the `pr` skill's verdict `통과` for the PR's current head
SHA. Confirm the SHA with `gh pr view <number> --json headRefOid`; do not
re-derive gate results yourself.

- `통과` for an older SHA, or no verdict: send the worker to rerun the `pr`
  skill for the current head.
- A local gate (`command` or `agent-review`) failed before push, so no PR or
  verdict exists: send the fix to the worker. Rerunning `pr` without a fix
  fails the same way.
- An `agent-review` gate names a `reviewer` the worker cannot start (Codex
  from a Claude worker, for example): start that reviewer as a separate,
  read-only Orca worker on the worker's branch and head SHA, give it the
  gate's inputs as the `pr` skill lists them, and send its findings to the
  implementing worker as that gate's result.
- `실패`: send the fixes from the verdict's diagnosis to the worker, then wait
  for a new verdict.
- `미완료`: wait for the worker to rerun the `pr` skill's wait; if it stays
  pending, report the pending gate. Never manufacture a completion signal.
- `검증 신호 없음`: nothing was verified. Do not merge on your own; once the
  rest of step 5 is clear, give the user the PR URL and SHA and ask whether
  to merge.

The verdict is the minimum floor, not an instruction to ignore unresolved
review findings, merge conflicts, unmet requirements, or user-requested
validation. Those block merge in every project, whatever the gate set. A
material risk is any security, authorization, data-loss, or money-handling
concern found in the diff or reviews.

### 5. Decide Whether to Merge

Use this decision tree:

```text
Is the PR head SHA unchanged since validation?
├── No → invalidate prior gates and wait for current-SHA results
└── Yes
    ├── `pr` verdict for this SHA?
    │   ├── `실패`, `미완료`, or none → do not merge
    │   └── `통과` or `검증 신호 없음` → continue
    ├── Any unresolved finding, conflict, unmet requirement, or material risk?
    │   └── Yes → do not merge; send a fix to a worker or report the blocker
    ├── Verdict was `검증 신호 없음`?
    │   └── Yes → ask the user; merge only on explicit approval for this SHA
    └── Otherwise → the coordinator may merge
```

A `통과` verdict permits merge consideration; it never forces a merge.

Merge with `gh pr merge <number> --squash --delete-branch`. After merging,
verify the PR state and merge commit, then tear the workspace down (step 6).
The worker's worktree still has the branch checked out, so the command may
exit non-zero only because local branch deletion failed. Confirm the merged
state with `gh pr view <number> --json state,mergeCommit` instead of retrying
the merge; the worktree removal in step 6 releases the branch.

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
withheld decision, exact SHA, the `pr` verdict and each gate's result, what was
handed off, and any remaining risk.
