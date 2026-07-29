---
name: pr
description: Prepare and create a non-draft GitHub pull request by verifying the current branch, resolving or confirming the intended base, summarizing the actual diff, honoring the repository PR template, pushing, and opening the PR. Use when the user asks to create, open, push, or publish a pull request, or invokes `$pr`.
---

# Pull Request

Create a review-ready PR whose base and contents are intentional. Do not
auto-commit work or invent repository conventions.

## Workflow

1. Inspect `git status --short`, `git branch -vv`, `git remote -v`, and the
   current branch. Stop for a detached HEAD or an empty branch.
2. Require a clean worktree unless the user explicitly authorizes publishing
   work in progress. Do not invoke the commit workflow implicitly.
3. Resolve the base:
   - Treat an explicit `$pr <branch>` argument or a base named in the request as
     confirmed.
   - Verify that the explicit ref exists locally or on the remote.
   - Otherwise inspect the upstream, remote default branch, ancestry, recent
     graph, commit ranges, and diff sizes.
   - Detect stacked work by looking for a parent feature branch that is an
     ancestor of `HEAD` or produces the smallest coherent diff.
4. If the base was inferred, present the best candidate, alternatives, commit
   count, and diff stat, then ask once for confirmation before pushing.
5. Inspect `<base>...HEAD` directly. Build the summary from source diffs rather
   than commit subjects alone.
6. Read `.github/PULL_REQUEST_TEMPLATE.md` or other repository-local PR
   templates when present. Preserve their headings and follow repository
   language and formatting conventions.
7. Derive a concise reviewer-facing title. If the branch contains a ticket key
   such as `ABC-123`, prefix the title with `[ABC-123]`; otherwise omit a ticket
   prefix. Do not invent one.
8. Push the current branch with upstream when needed, then create a non-draft PR
   with `gh pr create --base <base> --head <branch> --title <title>
   --body-file <file>`.
9. Report the PR URL, base, head, pushed commit range, and remaining local
   status.

If GitHub CLI is missing or unauthenticated, report the prepared title, body,
and exact command instead of claiming publication succeeded.

## Visual evidence

Request screenshots only when the diff changes rendered layout, styling,
content hierarchy, interaction states, or a visual defect reviewers need to
compare. Skip them for non-visual refactors, tests, docs, generated files,
dependency changes, or configuration-only changes.

Before capturing or uploading anything:

1. Propose the minimum useful views with route or component, state, viewport
   when relevant, and what each image proves.
2. Ask the user to select all, specific views, or skip. Combine this with base
   confirmation when both are pending.
3. Capture only approved views using non-sensitive data.
4. Upload attachments without committing temporary evidence and verify that the
   PR body renders them.

An explicit base confirms only the base, not screenshot upload.

## Base decision rules

Prefer the base that yields the smallest coherent review unit:

- Use a parent feature branch for clearly stacked work.
- Use the repository's evidenced integration or default branch for independent
  work.
- Treat common branch names such as `main`, `master`, or `develop` as
  candidates, never assumptions.
- Ask with concrete diff and commit counts when two candidates remain plausible.

Do not push or create the PR until every required confirmation is resolved.
