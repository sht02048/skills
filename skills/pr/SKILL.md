---
name: pr
description: Prepare and create a non-draft GitHub pull request by verifying the current branch, resolving or confirming the intended base, summarizing the actual diff, honoring the repository PR template, pushing, opening the PR, and then verifying it against the project's merge gates (`.agents/pr-gates.json`: project commands such as React Doctor, an independent agent review, CI checks, the automatic Codex review) and reporting a verdict for the head SHA. Also re-verifies an existing PR after new commits, and adds, removes, or changes the project's gates on request. Use when the user asks to create, open, push, or publish a pull request, or to change how PRs are verified; says "PR 올려줘", "PR 만들어줘", "PR 게이트 추가", or "PR 검증 바꿔줘"; or invokes `$pr`; other skills such as `orca-orchestration` call it to get a merge-ready verdict.
---

# Pull Request

Create a review-ready PR whose base and contents are intentional, then verify
it against the project's merge gates. Do not auto-commit work, invent
repository conventions, or merge. Do not push or create the PR until every
required confirmation is resolved.

## Workflow

1. Inspect `git status --short`, `git branch -vv`, `git remote -v`, and the
   current branch. Stop for a detached HEAD or an empty branch.
2. Stop and report uncommitted changes unless the user explicitly authorizes
   publishing work in progress; they would not be in the PR, so its summary
   would misdescribe the working tree. Do not invoke the commit workflow
   implicitly. An uncommitted `.agents/pr-gates.json` alone does not block;
   leave it out of the PR and mention it in the report.
3. Resolve the merge gates (see "Merge gates"). If the user must be asked,
   hold the question and ask it in step 6 together with base confirmation;
   ask it alone when no base question is pending.
4. Check for an open PR from this branch with
   `gh pr view --json number,url,headRefOid,baseRefName`. If one exists, this
   run re-verifies it: skip steps 5–7 and 9–10, run step 8 on the new HEAD,
   push without creating a PR in step 11, then go to step 13.
5. Resolve the base:
   - Treat an explicit `$pr <branch>` argument or a base named in the request as
     confirmed.
   - Verify that the explicit ref exists locally or on the remote.
   - Otherwise inspect the upstream, remote default branch, ancestry, recent
     graph, commit ranges, and diff sizes, and apply "Base decision rules".
6. If the base was inferred, present the best candidate, alternatives, commit
   count, and diff stat, then ask once for confirmation before pushing.
7. Inspect `<base>...HEAD` directly. Build the summary from source diffs rather
   than commit subjects alone.
8. Run every local gate, `command` and `agent-review` (see "Run local
   gates"). If one fails, stop before pushing and report it, unless the user
   or the calling task explicitly asks to open the PR anyway.
9. Read `.github/PULL_REQUEST_TEMPLATE.md` or other repository-local PR
   templates when present. Preserve their headings and follow repository
   language and formatting conventions. Without a template or convention, write
   a Korean title and a Korean body with exactly these sections, in order:
   - `## 배경`: the ticket link and its requirement in one or two lines when
     a ticket exists, and the problem this PR solves.
   - `## 요약`: the changed behavior, taken from the diff and described by
     behavior rather than as a file list.
   - `## 결정`: the approach chosen, the alternatives dropped, and why;
     include judgment calls the ticket did not specify.
   - `## 범위 밖 · 남은 위험`: what was deliberately left untouched, known
     limitations, and candidate follow-up tickets.
   - `## 검증`: what was run, including each local gate and its result, and
     what was not, naming the commit SHA the results apply to.

   Read `references/body.md` before writing this body; it holds good and bad
   examples per section and a full example.
10. Write a one-line title that states the change, not the branch name. If the
    branch contains a ticket key such as `ABC-123`, prefix the title with
    `[ABC-123]`; otherwise omit a ticket prefix. Do not invent one.
11. Push the current branch with upstream when needed, then create a non-draft
    PR with `gh pr create --base <base> --head <branch> --title <title>
    --body-file <file>`.
12. Report the PR URL, base, head, pushed commit range, and remaining local
    status immediately, without waiting for checks.
13. Follow "Wait for checks and review" for the PR's head SHA, then report the
    verdict (see "Verdict").

If GitHub CLI is missing or unauthenticated, report the prepared title, body,
and exact command instead of claiming publication succeeded.

## Merge gates

Projects differ in which checks they can run: React Doctor fits only React
code, and the automatic Codex review exists only where the Codex GitHub
integration is enabled. Each project lists its gates in `.agents/pr-gates.json`
at the repository root, committed so every agent and teammate verifies against
the same set. CI checks reported on the PR are always part of the verdict and
need no entry.

```json
{
  "version": 1,
  "waitMinutes": 10,
  "gates": [
    { "type": "command", "name": "React Doctor", "run": "npx -y react-doctor@latest . --yes" },
    { "type": "command", "name": "Tests", "run": "pnpm test" },
    {
      "type": "agent-review",
      "name": "독립 리뷰",
      "instructions": "Find correctness bugs, security issues, and unmet requirements. Ignore style.",
      "reviewer": "codex"
    },
    { "type": "codex-auto-review" }
  ]
}
```

| Gate type | Fields | Passes when |
| --- | --- | --- |
| `command` | `name`, `run` | `run` exits 0 from the repository root on the head SHA |
| `agent-review` | `name`, `instructions`, optional `reviewer` | An independent agent reviews `<base>...HEAD` at the head SHA against `instructions` and reports no blocking finding |
| `codex-auto-review` | none | The automatic Codex review finishes with 👍 for the head SHA |

`waitMinutes` is optional: how long "Wait for checks and review" waits, a
whole number of minutes from 1 to 30. Without it, wait 10 minutes.

An empty `gates` list is valid: only CI checks count. The file is invalid when
`version` is not `1`, `gates` is missing, `waitMinutes` is present but not a
whole number from 1 to 30, a gate has an unknown `type`, a
`command` gate lacks `name` or `run`, or an `agent-review` gate lacks `name`
or `instructions`. Never guess a repair.

These three types cover any check a user wants. A check a program can decide
is a `command`; a check that needs judgment, such as a security review, an
accessibility pass, or conformance to a design doc, is an `agent-review` with
those criteria in `instructions`. Several gates of the same type are fine.

Resolve the gate set in this order:

1. When the caller provides a resolved gate set (an Orca task spec, for
   example), use it and do not ask.
2. Otherwise read `.agents/pr-gates.json`. If it is valid, use it and name the
   gates in the first report.
3. If the file is missing or invalid, inspect the repository so the choices
   rest on evidence:
   - React project: `react` appears in a `package.json` dependency list.
   - Automatic Codex review: recent PRs carry reviews or reactions from the
     Codex bot account.
   - Project verification commands: scripts the project already defines, such
     as `test`, `lint`, or `typecheck` in `package.json`, or the equivalents
     in its build tool.
4. Ask with the host's structured question tool (AskUserQuestion in Claude
   Code; otherwise one message with numbered options). Ask these questions
   together, combined with base confirmation when that is also pending:
   - Which gates to use (multi-select): React Doctor, the automatic Codex
     review, a project verification command, and an independent agent
     review. Mark the options the inspection supports as recommended and say
     what was detected. Put the detected command in the command option's
     text. The free-form answer covers anything else: map it to a `command`
     or an `agent-review` and confirm the resulting gate with the user; if it
     fits neither, say so instead of inventing a type. When the file was
     invalid, state the exact problem.
   - When an independent agent review is selected without criteria, use
     "Find correctness bugs, security issues, and unmet requirements. Ignore
     style." and leave `reviewer` unset.
   - How long to wait for checks and the Codex review: 10 minutes
     (recommended), 20 minutes, or 30 minutes. A free-form answer must be a
     whole number of minutes from 1 to 30; cap a longer one at 30 and say
     so. Save 10 by omitting `waitMinutes`.
   - Whether to save the choice as project configuration (recommended) or use
     it for this run only.
5. To save, write `.agents/pr-gates.json` after the PR is created so it does
   not enter this PR, and report it as an uncommitted file for the user to
   commit separately. A caller that forbids file edits (the Orca coordinator)
   saves it through its own worker instead.

React Doctor is a `command` gate. Use the project's own script for it when one
exists; otherwise use `npx -y react-doctor@latest . --yes`. The trailing
`--yes` skips React Doctor's workspace prompt, which would otherwise hang a
non-interactive run. It exits non-zero on error-level diagnostics. When the
repository already has error-level diagnostics, offer
`npx -y react-doctor@latest . --yes --scope changed` so the gate judges only
issues the PR introduces.

## Change the gates

When the user asks to add, remove, or change a gate or the wait time, read
the current `.agents/pr-gates.json` (or start from
`{"version": 1, "gates": []}`), map the request to a gate or a `waitMinutes`
value as in "Merge gates", show the resulting JSON, and write it after the
user confirms. Validate it against the rules above. Report it as an
uncommitted file for the user to commit; do not commit it implicitly. A caller
that forbids file edits (the Orca coordinator) has a worker make the change.

## Run local gates

Run local gates on the exact HEAD that will be pushed. A result for another
SHA is stale; rerun every local gate after any new commit.

**`command`.** Run it from the repository root. Record the command, exit
status, and SHA, and keep the output's failing lines for the report. Remove
any output the command leaves in the working tree that is not ignored by
Git, and confirm `git status --short` is unchanged.

**`agent-review`.** The reviewer must be an agent that did not write the
change, starting from a fresh context:

- Give it only the repository path, the base, the head SHA, the change's
  stated goal (the ticket or PR summary), and the gate's `instructions`. Do
  not pass the author's reasoning or your own verdict; a reviewer handed a
  conclusion tends to confirm it.
- Ask for findings with file, line, the failure they cause, and whether each
  one blocks merge under the `instructions`. The reviewer reads and reports;
  it never edits.
- Start it with what the host provides: a fresh subagent where the host has
  one, or the agent named by `reviewer` (such as `codex`). If the named
  reviewer cannot be started here, report the gate as not run instead of
  substituting another agent; the verdict cannot pass. When the caller
  supplies that reviewer's result for the same head SHA (an Orca coordinator
  running it as a separate worker), use it as the gate's result.
- Check each blocking finding against the code before counting it. Discard
  one only when the code plainly contradicts it, and say why in the report.
- Record the reviewer, SHA, blocking and non-blocking findings. The gate
  passes when no blocking finding remains for this SHA. Fixes need a new
  commit, which makes every local gate rerun.

## Base decision rules

Prefer the base that yields the smallest coherent review unit:

- Use a parent feature branch when it is an ancestor of `HEAD` and the diff
  against it is smaller than against the default branch.
- Use the repository's evidenced integration or default branch for independent
  work.
- Treat common branch names such as `main`, `master`, or `develop` as
  candidates, never assumptions.
- Ask with concrete diff and commit counts when two candidates both satisfy
  these rules.

## Visual evidence

Request screenshots only when the confirmed-base diff changes rendered layout,
styling, content hierarchy, interaction states, or a visual defect reviewers
need to compare. Skip them for non-visual refactors, tests, docs, generated
files, dependency or configuration changes, and copy-only changes without a
review-relevant visual result. When skipped, do not ask about screenshots.

When screenshots are warranted:

1. Before starting a dev server, capturing, or uploading, propose a numbered
   candidate list. For each candidate include:
   - route or component;
   - state to reproduce, such as default, populated, error, or open dialog;
   - viewport when it matters;
   - the changed behavior the image proves.
2. Ask the user to select all candidates, specific numbers, or skip. Combine
   this with base confirmation when both decisions are pending. An explicit
   base confirms only the base, not the screenshot selection.
3. Capture only the selected views using mock or non-sensitive data. Capture one
   image per reviewer question, cropped to the changed area, without
   development overlays, secrets, or personal data.
4. Upload images as GitHub user attachments without committing temporary
   evidence:
   - Prefer an already-installed `gh-image` or `gh-attach` extension. If an
     extension must be installed, include that external change in the user's
     confirmation.
   - With `gh-image`, run
     `gh image --repo <owner>/<repo> <screenshot-path>` and capture the returned
     Markdown. Do not expose session tokens in arguments or logs.
   - Replace the filename-only alt text with a description of what the
     screenshot proves, then insert the Markdown into the PR body.
5. After creating the PR, inspect its rendered body with
   `gh pr view <number> --json body` and verify each attachment URL returns a
   successful HTTP response. Remove temporary screenshots and PR body files,
   then confirm they were not committed with `git status --short`.

## Wait for checks and review

After reporting the PR, wait up to the wait time in total for the checks and,
when `codex-auto-review` is a gate, the automatic Codex review of the PR's head
SHA. Run the wait as a background task when the agent supports background
commands that resume it on completion, so the conversation stays usable;
otherwise wait in the foreground.

The wait time is, in order: a time the user or the calling task names for this
run, then `waitMinutes` from the resolved gate set, then 10 minutes. It is
never more than 30 minutes; cap a longer request at 30 and say so. Name the
wait time in the report.

Poll every 30 seconds until every signal settles or the wait time passes:

- Checks: `gh pr checks <number> --json bucket,name,state,link`. Checks are
  settled when none is `pending`. If no check appears within the first two
  minutes, treat the repository as having no checks; CI often registers its
  checks a minute or more after the PR opens.
- Codex review (only when it is a gate): read reactions on the PR with
  `gh api "repos/{owner}/{repo}/issues/<number>/reactions"` and reviews with
  `gh api "repos/{owner}/{repo}/pulls/<number>/reviews"`, considering only the
  Codex bot account. 👀 means the review is still running. 👍 means it finished
  with no blocking finding. A submitted Codex review for the head SHA means it
  finished with feedback to read. Reactions carry no commit SHA, so count a
  👀 or 👍 only when its `created_at` is later than this run's push of the
  head SHA (or, when this run pushed nothing, the head commit's committer
  date); an older reaction belongs to an earlier commit. Count a review only
  when its `commit_id` is the head SHA. Because the project declared this
  gate, a missing signal means pending, never "not configured".

Never post `@codex review`, rerun workflows, or otherwise trigger a check or
review manually; opening or updating the PR already starts the configured
automation, and a manual trigger duplicates it.

## Verdict

Bind the verdict to the head SHA the gates ran against. Report it with the PR
URL, head SHA, and each gate's result, using exactly one of these:

- `통과`: every local gate passed on this SHA, every existing check passed,
  and Codex gave 👍 when it is a gate.
- `실패`: a local gate or check failed or was cancelled, an `agent-review`
  gate could not run, or Codex submitted a review instead of 👍. Follow the `pr-review-diagnosis` skill for this PR
  and report its diagnosis. Do not edit files, commit, push, or reply on
  GitHub.
- `미완료`: the wait time passed with something still unsettled. Name what is
  still pending. If something has already failed, report `실패` instead.
- `검증 신호 없음`: the gate set is empty and no check exists. State that no
  automated verification ran. Never call this a pass; nothing was verified.

Only `통과` is a pass, and only for the SHA it names. Any new commit makes it
stale.
