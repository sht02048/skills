---
name: commit
description: Create focused Git commits from the current repository changes using a staged-first policy, concern-based splitting, and Korean gitmoji commit messages. Use when the user asks to commit, split changes into commits, says "커밋", "커밋해줘", or "나눠서 커밋", or invokes `$commit`.
---

# Commit

Never commit, stage, unstage, or rewrite more than the user authorized.

## Hard constraints

- Never run `git add .`, `git add -A`, `git commit -a`, or `git commit <path>`
  unless every visible change belongs to the same commit; they sweep in
  unrelated hunks, untracked files, and secrets, and break partial staging.
- Never stage files that look like secrets (`.env*`, `*.pem`, `*.key`,
  credential or token files). List them in the report instead.
- When a hook fails, report its output and stop. Never pass `--no-verify`; it
  skips the repository's own quality gates.

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

Commit only the staged snapshot, even when it exceeds the split trigger below.
In that case, commit it as one and report its size with a suggested split.

## Build commits when nothing is staged

1. Inspect tracked and untracked changes with `git status --short`, `git diff`,
   and `git ls-files --others --exclude-standard`.
2. Group files by one reviewer-visible concern.
3. Stage each group with explicit pathspecs.
4. Inspect the staged diff before committing.
5. Re-run `git status --short` after every commit.

Split a group into its own commit when it is correct without the other groups
and its subject can be written without `및` or `and`. Keep tests with the
behavior they protect so each commit passes on its own and reverts cleanly.
Keep one commit when splitting would create broken or misleading intermediate
states.

Treat 20 or more files, 500 or more changed lines, or three unrelated top-level
areas as a split trigger. Derive smaller coherent slices before committing.

If there are no changes, report that there is nothing to commit.

## Write the message

Use this subject format:

```text
<emoji> <type>: <Korean action-noun title>
```

Keep the Korean title at 30 characters or fewer and end it with an action
noun such as `구현`, `정리`, `추가`, `수정`, `삭제`, `기록`, `분리`, `개선`,
`갱신`, or `보강`.

- Bad: `✨ feat: 로그인 기능을 추가했습니다`
- Good: `✨ feat: 로그인 폼 검증 추가`

Write a Korean body only when the reason for the change is not visible in the
diff, the commit contains a breaking change, or files were grouped for a
non-obvious reason. Preserve repository-required trailers exactly.

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

When a commit spans several types, pick by precedence `fix` > `feat` >
`refactor` > the rest; tests and docs inside a behavior commit do not change
its type. A change to agent instructions such as `SKILL.md` is `feat` when it
changes agent behavior and `docs` when it changes wording or names only.

## Report

Report each commit hash and subject plus any remaining working-tree changes and
skipped secret-like files. If the staged-first path was used, explicitly state
that unstaged changes were left untouched.
