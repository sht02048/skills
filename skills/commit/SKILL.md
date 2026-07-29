---
name: commit
description: Create focused Git commits from the current repository changes using a staged-first policy, concern-based splitting, and Korean gitmoji commit messages. Use when the user asks to commit, split changes into commits, or invokes `$commit`.
---

# Commit

Create safe, reviewable commits from repository evidence. Never commit, stage,
unstage, or rewrite more than the user authorized.

## Preserve an existing staged snapshot

When staged changes exist, treat the index as the user's selected commit
artifact. Inspect it with:

- `git status --short`
- `git diff --cached --stat`
- `git diff --cached`
- `git diff --check --cached`

Do not edit files, run mutating formatters or generators, stage, unstage,
restore, or otherwise change the snapshot unless the user explicitly requests
that change. If validation finds a problem, report it instead of repairing the
staged content.

Commit only the staged snapshot. Preserve partial staging when staged and
unstaged changes coexist in one file.

## Build commits when nothing is staged

1. Inspect tracked and untracked changes with `git status --short`, `git diff`,
   and `git ls-files --others --exclude-standard`.
2. Group files by one reviewer-visible concern.
3. Stage each group with explicit pathspecs. Avoid `git add .` unless every
   visible change belongs to the same commit.
4. Inspect the staged diff before committing.
5. Re-run `git status --short` after every commit.

Split documentation, tooling, refactors, behavior changes, and unrelated
application areas when they can stand alone. Keep tests with the behavior they
protect. Keep one commit when splitting would create broken or misleading
intermediate states.

Treat more than 20 files, roughly 500 changed lines, or three unrelated
top-level areas as a review trigger. Derive smaller coherent slices before
committing unless the user explicitly asks to preserve the staged snapshot.

If there are no changes, report that there is nothing to commit.

## Write the message

Use this subject format:

```text
<emoji> <type>: <Korean action-noun title>
```

Keep the title concise and concrete. Prefer nouns such as `구현`, `정리`,
`추가`, `수정`, `삭제`, `기록`, `분리`, `개선`, `갱신`, or `보강`. Write a
Korean body only when it adds useful context. Preserve repository-required
trailers exactly.

Use this mapping:

- `✨ feat`: 기능 작업
- `♻️ refactor`: 동작을 유지한 내부 구조 개선
- `🐛 fix`: 버그 수정
- `📝 docs`: 문서 변경
- `💄 style`: 동작과 무관한 코드 포맷 변경
- `🎨 design`: 동작과 무관한 시각 스타일 변경
- `🌐 i18n`: 번역 리소스 변경
- `✅ test`: 테스트 변경
- `🔧 config`: 프로젝트 또는 환경 설정 변경
- `🎸 chore`: 그 밖의 유지보수 작업

## Report

Report each commit hash and subject plus any remaining working-tree changes. If
the staged-first path was used, explicitly state that unstaged changes were left
untouched.
