---
name: pr-review-diagnosis
description: Fetch and assess every review attached to the pull request for the current Git branch, classifying whether each substantive thread is valid, already resolved, inapplicable, or blocked on missing facts. Use when the user asks to inspect, triage, assess, or diagnose PR reviews or invokes `$pr-review-diagnosis`.
---

# PR Review Diagnosis

Diagnose only. Do not edit files, resolve conversations, reply on GitHub,
commit, or push unless the user explicitly asks.

## Workflow

1. Confirm the repository and current branch with `git status --short` and
   `git branch --show-current`. Stop for a detached HEAD.
2. Resolve the current branch's open PR with:
   `gh pr view --json number,url,title,baseRefName,headRefName,reviewDecision,reviews,comments`.
3. Fetch paginated inline comments with:
   `gh api --paginate "repos/{owner}/{repo}/pulls/<number>/comments"`.
4. Include submitted reviews and general PR comments. Preserve author, body,
   path, line, commit SHA, timestamps, URL, reply relationships, and
   outdated/resolved state when available. Group replies into threads.
5. Inspect the current `<base>...HEAD` diff, referenced code, relevant callers,
   contracts, and tests before judging a comment. Check whether later commits
   already addressed it.
6. Classify every substantive thread into exactly one category:
   - `조치 필요`: valid and actionable on current `HEAD`
   - `확인 필요`: a specific missing product, contract, security, or ownership
     fact prevents a decision
   - `타당하지만 이미 해결`: valid at the reviewed revision but resolved now
   - `오탐·비적용`: contradicted by current evidence or outside the PR's actual
     scope
7. Report in Korean in the category order above. Include reviewer, location,
   rationale, current evidence, smallest recommended response, and comment URL.
   Separate automated and human feedback in the summary.

Ignore empty approval-only reviews. Judge comments by evidence, not by whether
they are unresolved, marked as requested changes, automated, or outdated. If
nothing requires action, say so and name any residual uncertainty.

Never expose credentials, tokens, or private repository data.
