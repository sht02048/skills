---
name: pr-review-diagnosis
description: Diagnose reviews and failed checks for the pull request on the current Git branch, classifying review threads and determining whether GitHub Actions or other CI checks failed on the current head commit. Use when the user asks to inspect, triage, or diagnose a PR or its reviews and checks, says "PR 리뷰 봐줘", "리뷰 확인", or "CI 왜 실패했어", or invokes `$pr-review-diagnosis`.
---

# PR Review Diagnosis

Diagnose only. Do not edit files, resolve conversations, reply on GitHub,
commit, or push unless the user explicitly asks.

## Workflow

1. Confirm the repository and current branch with `git status --short` and
   `git branch --show-current`. Stop for a detached HEAD.
2. Resolve the current branch's open PR with:
   `gh pr view --json number,url,title,baseRefName,headRefName,headRefOid,reviewDecision,reviews,comments`.
   Stop and report when no open PR exists or GitHub CLI is unauthenticated.
3. Fetch every check for that PR with:
   `gh pr checks <number> --json bucket,state,name,workflow,description,link,startedAt,completedAt`.
   Evaluate checks for the reported `headRefOid`, not an older run. Treat `fail`
   as failed, `cancel` as cancelled, and `pending` as unfinished; none is a pass.
   For each failed GitHub Actions check, inspect its run summary and failed-step
   log with `gh run view <run-id> --json status,conclusion,jobs,url` and
   `gh run view <run-id> --log-failed`. Do not rerun workflows.
4. Fetch paginated inline comments with:
   `gh api --paginate "repos/{owner}/{repo}/pulls/<number>/comments"`.
5. Include submitted reviews and general PR comments. Preserve author, body,
   path, line, commit SHA, timestamps, URL, reply relationships, and
   outdated/resolved state when available. Group replies into threads.
6. Inspect the current `<base>...HEAD` diff, referenced code, relevant callers,
   contracts, and tests before judging a comment. Check whether later commits
   already addressed it.
7. Classify every substantive thread into exactly one category. A thread is
   substantive when it asserts a defect, requests a change, or asks a question;
   empty approvals and bot status messages without findings are not.
   - `조치 필요`: valid and actionable on current `HEAD`
   - `확인 필요`: a specific missing product, contract, security, or ownership
     fact prevents a decision
   - `타당하지만 이미 해결`: valid at the reviewed revision but resolved now
   - `오탐·비적용`: contradicted by current evidence or outside the PR's actual
     scope

Judge comments by evidence, not by whether they are unresolved, marked as
requested changes, automated, or outdated.

Never expose credentials, tokens, or private repository data.

## Report

Report in Korean with this exact shape. Omit empty category sections.

```markdown
## 체크

- head: `<headRefOid>`
- 결과: 통과 | 실패 N건 | 미완료 N건
- `<check name>` (실패|취소|미완료): evidence-backed cause from the failed-step log, <link>

## 조치 필요

- **Thread summary** — reviewer (자동|사람), `path:line`
  - 근거: current evidence for the classification
  - 권장 대응: smallest recommended response
  - <comment URL>

## 요약

자동 N건, 사람 N건 / 조치 필요 N, 확인 필요 N, 이미 해결 N, 오탐·비적용 N
```

Order category sections as listed in step 7. If nothing requires action, say so
in `## 요약` and name any residual uncertainty.
