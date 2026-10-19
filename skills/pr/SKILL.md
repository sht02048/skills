---
name: pr
description: Prepare and create a non-draft GitHub pull request by verifying the current branch, resolving or confirming the intended base, summarizing the actual diff, honoring the repository PR template, pushing, opening the PR, and then waiting for its checks and automatic Codex review to finish. Use when the user asks to create, open, push, or publish a pull request, says "PR 올려줘" or "PR 만들어줘", or invokes `$pr`.
---

# Pull Request

Create a review-ready PR whose base and contents are intentional. Do not
auto-commit work or invent repository conventions. Do not push or create the PR
until every required confirmation is resolved.

## Workflow

1. Inspect `git status --short`, `git branch -vv`, `git remote -v`, and the
   current branch. Stop for a detached HEAD or an empty branch.
2. Stop and report uncommitted changes unless the user explicitly authorizes
   publishing work in progress; they would not be in the PR, so its summary
   would misdescribe the working tree. Do not invoke the commit workflow
   implicitly.
3. Resolve the base:
   - Treat an explicit `$pr <branch>` argument or a base named in the request as
     confirmed.
   - Verify that the explicit ref exists locally or on the remote.
   - Otherwise inspect the upstream, remote default branch, ancestry, recent
     graph, commit ranges, and diff sizes, and apply "Base decision rules".
4. If the base was inferred, present the best candidate, alternatives, commit
   count, and diff stat, then ask once for confirmation before pushing.
5. Inspect `<base>...HEAD` directly. Build the summary from source diffs rather
   than commit subjects alone.
6. Read `.github/PULL_REQUEST_TEMPLATE.md` or other repository-local PR
   templates when present. Preserve their headings and follow repository
   language and formatting conventions. Without a template or convention, write
   a Korean title and a Korean body with exactly `## 요약` and `## 검증`
   sections; `## 검증` states what was run and what was not.
7. Write a one-line title that states the change, not the branch name. If the
   branch contains a ticket key such as `ABC-123`, prefix the title with
   `[ABC-123]`; otherwise omit a ticket prefix. Do not invent one.
8. Push the current branch with upstream when needed, then create a non-draft PR
   with `gh pr create --base <base> --head <branch> --title <title>
   --body-file <file>`.
9. Report the PR URL, base, head, pushed commit range, and remaining local
   status immediately, without waiting for checks.
10. Follow "Wait for checks and review" for the created PR.

If GitHub CLI is missing or unauthenticated, report the prepared title, body,
and exact command instead of claiming publication succeeded.

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

After reporting the PR, wait up to 10 minutes in total for the checks and the
automatic Codex review of the PR's head SHA. Run the wait as a background task
when the agent supports background commands that resume it on completion, so
the conversation stays usable; otherwise wait in the foreground.

Poll every 30 seconds until both signals settle or 10 minutes pass:

- Checks: `gh pr checks <number> --json bucket,name,state,link`. Checks are
  settled when none is `pending`. If no check appears within the first two
  minutes, treat the repository as having no checks; CI often registers its
  checks a minute or more after the PR opens.
- Codex review: read reactions on the PR with
  `gh api "repos/{owner}/{repo}/issues/<number>/reactions"` and reviews with
  `gh api "repos/{owner}/{repo}/pulls/<number>/reviews"`, considering only the
  Codex bot account. 👀 means the review is still running. 👍 means it finished
  with no blocking finding. A submitted Codex review for the head SHA means it
  finished with feedback to read. If no Codex signal appears within the first
  two minutes, treat automatic review as not configured and stop waiting for
  it.

Never post `@codex review`, rerun workflows, or otherwise trigger a check or
review manually; opening the PR already starts the configured automation, and a
manual trigger duplicates it.

When the wait ends:

- No check exists and Codex is not configured: report `검증 신호 없음` with
  the head SHA, stating that no automated verification ran. Never call this a
  pass; nothing was verified.
- Otherwise, every existing check passed and Codex gave 👍 or is not
  configured: report the pass with the head SHA.
- Any check failed or was cancelled, or Codex submitted a review instead of
  👍: follow the `pr-review-diagnosis` skill for this PR and report its
  diagnosis. Do not edit files, commit, push, or reply on GitHub.
- 10 minutes passed with something still unsettled: stop waiting, name what is
  still pending, and do not report it as a pass. Still run the diagnosis if
  something has already failed.
