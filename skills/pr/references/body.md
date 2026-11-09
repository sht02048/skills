# PR body guide

How to fill each section of the default body (`SKILL.md` step 9). A
repository PR template still wins; use this only when there is none.

## Contents

1. Principles
2. Sections
3. Full example

---

## 1. Principles

- Every claim must match `<base>...HEAD`. A body that describes code the diff
  does not contain misleads the reviewer and the coordinator alike.
- Write for three readers: a human reviewer, an automated reviewer (an
  `agent-review` gate or Codex), and the next agent that picks up a fix after
  the worktree is gone. The body is the record that outlives the workspace.
- Skip preambles. Use short sentences, lists, and the project's domain terms
  (from `GLOSSARY.md` or similar when present).
- Leave agent internals out: worker prompts, Task or Dispatch IDs, raw logs.

## 2. Sections

### 배경

The ticket link, its requirement in one or two lines, and the problem.
Without a ticket, state the problem only; never invent a ticket key.

- Good: `Closes #123. 목록을 새로고침하면 필터가 초기화돼서, 필터를 URL
  쿼리에 저장하도록 요청했어요.`
- Bad: `이 PR은 필터 관련 작업입니다.` (no requirement, no problem)

### 요약

The changed behavior, by behavior rather than by file. When a picture makes
the change clearer than prose, use the smallest one that does:

| Change | Visual |
| --- | --- |
| logic or an algorithm | pseudocode |
| runtime control flow | call tree |
| UI structure | component tree |
| file responsibility or a broad refactor | shallow file tree |
| interaction or data flow | Mermaid sequence diagram |
| a change to an existing shape | any of the above as a `diff` sketch |

Place each visual next to the sentence it supports, keep only the nodes that
matter, and usually use one. A small, obvious change needs none.

```diff
 loadList
   readFiltersFromUrl
+  if filters are missing
+    restore last filters from storage
   fetchItems(filters)
```

- Bad: a list of changed files, or commit subjects pasted in order.

### 결정

The approach chosen, the alternatives dropped, and why. Name every judgment
call the ticket did not specify, so a reviewer can disagree with it
explicitly. For a bug fix, state the hypothesis that proved correct.

- Good: `localStorage 대신 URL 쿼리를 썼어요. 필터가 걸린 화면을 링크로
  공유할 수 있어야 해서예요.`
- Bad: `최선의 방법으로 구현했어요.`

### 범위 밖 · 남은 위험

What was deliberately left untouched, known limitations, and candidate
follow-up tickets. Write `없음` only after checking; an empty section is a
claim too.

- Good: `정렬 상태는 이번 티켓 범위가 아니라 그대로 뒀어요. URL 길이 제한을
  넘는 필터 조합은 확인하지 않았어요.`

### 검증

What was run and its result, and what was not, against a named commit SHA.
Prefer evidence a reviewer can check:

- Visual change: a screenshot (see "Screenshots" in `SKILL.md`).
- Behavior change: the test that failed before the fix and passes after,
  named or sketched, with the command run.
- Each local gate from `.agents/pr-gates.json` and its result.
- What was not run, and why.

- Bad: `테스트 통과했어요.` (which tests, on which commit)

## 3. Full example

````markdown
## 배경
Closes #123. 목록을 새로고침하면 필터가 초기화돼서, 필터를 URL 쿼리에
저장하도록 요청했어요.

## 요약
필터를 바꾸면 URL 쿼리에 쓰고, 목록을 열 때 쿼리에서 필터를 읽어요.

```diff
 loadList
-  filters = defaultFilters
+  filters = readFiltersFromUrl() ?? defaultFilters
   fetchItems(filters)
```

## 결정
- localStorage 대신 URL 쿼리를 썼어요. 필터가 걸린 화면을 링크로 공유할
  수 있어야 해서예요.
- 쿼리 이름은 기존 검색 쿼리와 겹치지 않게 `f_` 접두사를 붙였어요.
  티켓에 정해지지 않아 제가 고른 부분이에요.

## 범위 밖 · 남은 위험
- 정렬 상태는 이번 범위가 아니라 그대로 뒀어요.
- 필터가 많아 URL이 2,000자를 넘는 경우는 확인하지 않았어요.

## 검증
기준 커밋: `a1b2c3d`
- `filters persist across reload` 테스트: 수정 전 실패, 수정 후 통과
  (`pnpm test list-filters`)
- 로컬 게이트 `react-doctor`: 통과
- 실제 브라우저 새로고침은 확인하지 않았어요.
````

The visual menu in "요약" is adapted from Matt Pocock's `pr` skill, which
credits Dex Horthy's `show-me` skill.
