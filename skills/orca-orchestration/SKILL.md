---
name: orca-orchestration
description: Coordinate a requirement or ticket from an isolated Orca worktree through implementation, validation, pull request, and merge judgment. Use when the user asks to process a ticket or feature with Orca orchestration, create an Orca worktree for delegated implementation, keep the coordinator code-free, or merge only after React Doctor and the automatic Codex PR review pass.
---

# Orca Orchestration

Run the delivery through Orca's tracked worktree and orchestration state. The coordinator directs workers and decides gates; it never edits code.

## Non-Negotiable Ownership

- Create a dedicated Orca worktree for every requirement or ticket. Isolation keeps worker changes out of the coordinator checkout.
- Use Orca orchestration tasks and injected dispatches. A terminal prompt without a tracked task is not orchestration.
- Assign all repository mutations to workers: code, tests, generated files, formatting, commits, pushes, PR creation, and review fixes.
- Never let the coordinator run an editor, `apply_patch`, a formatter that writes files, `git commit`, or any equivalent code-changing command. The coordinator may inspect state, issue Orca/GitHub control commands, make decisions, and merge.
- Keep one explicit owner per file or responsibility when multiple workers run. Shared worktrees make overlapping edits unsafe.

## Delivery Flow

### 1. Establish the Worktree

1. Confirm Orca is running with `orca status --json`.
2. Decide lineage separately from the Git base:
   - Independent ticket: use `--no-parent` and the repository default base.
   - Stacked ticket: use an explicit parent or base only when the user requested that dependency.
3. Create the worktree with an agent in its first terminal:

```bash
orca worktree create --name <ticket>-<slug> --no-parent --agent codex --json
orca terminal wait --terminal <startupTerminal.handle> --for tui-idle --timeout-ms 60000 --json
```

Keep the full returned worktree ID and the single live terminal handle. Do not create a duplicate agent terminal.

### 2. Dispatch the Work

Create a task whose specification includes:

- the exact requirement or ticket source;
- repository instructions and scope boundaries;
- worker ownership of every code change;
- required tests and observable QA;
- commit, push, and PR creation;
- a final report containing the PR URL, exact head SHA, checks run, and unresolved risks.

```bash
orca orchestration task-create --spec "<complete worker task>" --json
orca orchestration dispatch --task <task_id> --to <startupTerminal.handle> --inject --json
orca orchestration dispatch-show --task <task_id> --json
```

The last command proves that orchestration provenance exists. Never describe untracked work as orchestrated.

### 3. Supervise Without Editing

Wait on lifecycle messages instead of repeatedly reading terminal output:

```bash
orca orchestration check --wait --types worker_done,escalation,decision_gate --timeout-ms 900000 --json
```

- Answer worker questions with evidence and judgment.
- On a defect or failed gate, dispatch the correction to a worker. The coordinator must not patch it.
- Treat a timeout as a checkpoint, not failure. Check task and terminal liveness, then continue waiting.
- Stop immediately when the user says to stop.

### 4. Evaluate the Pull Request

Bind every decision to the PR's current head SHA. A pass for an older SHA is stale after any new commit.

Minimum merge gates:

1. React Doctor completed successfully for the current head SHA.
2. The automatic Codex PR review completed and passed for the current head SHA.

Interpret the automatic Codex review on the PR:

```text
👀 on the automatic review item → review is still running; wait
👍 → review completed with no blocking finding
Neither 👀 nor 👍 on the automatic review item → Codex posted a separate review; fetch and read it before deciding
```

- A separate Codex review is a pass only when it reports no blocking issue for the current head SHA. The absence of reactions alone is never a pass. If the review contains findings, dispatch fixes to a worker and wait for the automatic review of the new SHA.
- Never post `@codex review`, call a review-triggering API, or otherwise start Codex review manually. Opening or updating the PR starts the configured automatic review.
- If no automatic completion signal appears, do not manufacture one and do not merge. Report the pending gate.

React Doctor and Codex are the minimum floor, not an instruction to ignore failed required CI, unresolved review findings, merge conflicts, unmet requirements, security or data risks, or user-requested validation.

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
    │   └── Yes → do not merge; dispatch a fix or report the blocker
    └── No → the coordinator may merge
```

Passing the minimum gates permits merge consideration; it never forces a merge. The coordinator must withhold merge when its evidence-based judgment finds material risk.

After merging, verify the PR state and merge commit. Report the worktree, PR, merged or withheld decision, exact SHA, React Doctor result, Codex review signal, and any remaining risk.
