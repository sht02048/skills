---
name: onboard
description: Brief a newcomer on an unfamiliar repository from repository evidence — purpose, stack, commands, structure, conventions, and pitfalls — optionally render a project-structure diagram with the `diagram-design` skill (installing it when missing), then deepen understanding through follow-up questions. Use when the user starts work in a new or unfamiliar repository, asks for onboarding, a repository overview, or how a codebase is organized, says "온보딩", "이 레포 설명해줘", or "프로젝트 구조 알려줘", or invokes `$onboard`.
---

# Onboard

Build an evidence-backed picture of the repository, then keep extending it
through follow-up questions. Never edit tracked files, install project
dependencies, or run project build, test, or dev commands; onboarding leaves the
repository as it was found, except for the optional diagram files below.

## Start

Before reading the repository, ask exactly one question: whether to generate a
project-structure diagram. Use `AskUserQuestion` when available. State in the
question that choosing it installs the `diagram-design` skill globally when it
is missing. Ask nothing else up front; every other choice has a default below.

## Evidence order

Resolve conflicts in this order:

1. Agent instructions: `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`,
   `.github/copilot-instructions.md`.
2. CI configuration such as `.github/workflows`. Commands CI runs are the ones
   known to pass; README commands are often stale.
3. Manifests and lockfiles. The lockfile decides the package manager
   (`pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `bun.lock*` → bun,
   `uv.lock` → uv); a workspaces declaration means a monorepo.
4. `README`, `CONTRIBUTING`, `docs/`, and `llm-wiki/`.
5. Git: `git log -20 --format=%s`, PR templates, and remote branch names.

For code structure, use `codegraph_explore` (or `codegraph explore`) when a
`.codegraph/` index exists. Never create or refresh an index; indexing belongs
to the user. Without one, read the directory tree two levels deep and the entry
files named by manifests.

## Briefing

Write in the user's language with this exact shape, shown for Korean:

```markdown
## 한 줄 요약

## 스택
- 언어·프레임워크:
- 패키지 매니저: (근거: lockfile)

## 명령어
- dev: `command` (출처: CI | scripts | README)
- build:
- test:
- lint:

## 구조
- `path/`: role
- 진입점: `path:line`

## 관례
- 커밋:
- PR:
- 코드 스타일:

## 함정

## 미확인

## 다음 질문
1. question (근거: `path`)
```

- Cite a path for every item. Never write a command you did not find in a
  source; list a missing command under `미확인`.
- Keep `구조` to eight entries or fewer, choosing the directories that own
  runtime behavior over tooling and generated output.
- Put conflicts between sources, required environment variables, and generated
  files under `함정`.
- Write exactly three `다음 질문`, each pointing at a specific ambiguity found
  while reading, such as two modules that appear to own the same concern. Never
  suggest a question the briefing already answers.

## Diagram

Run only when the user chose the diagram, after the briefing, because the
diagram draws the briefing's `구조`.

1. Locate `diagram-design`: check the available skill list, then `SKILL.md`
   under `~/.claude/skills/diagram-design`, `~/.agents/skills/diagram-design`,
   `~/.codex/skills/diagram-design`, and
   `~/.claude/plugins/cache/*/diagram-design`.
2. When missing, install it for the current host:
   `npx skills add cathrynlavery/diagram-design --global --agent <claude-code|codex> --skill diagram-design --yes`.
   If installation fails, report its output, draw an ASCII diagram in the
   conversation instead, and do not try other sources.
3. Read the installed `SKILL.md` and follow it directly; a skill installed
   mid-session may not appear in the skill list until restart.
4. Draw one Architecture diagram. Nodes are the entry points and modules from
   `구조`; edges are only calls or imports verified through CodeGraph or source.
   Omit unverified edges, because a plausible but wrong edge misleads more than
   a missing one.
5. At the `diagram-design` style gate, choose "keep default" without asking;
   onboarding diagrams explain structure, not brand. Never write a
   `.diagram-design` marker.
6. Save to `<repo>/.onboard/structure.html`, add `.onboard/` to
   `.git/info/exclude` so the file never appears in `git status`, and report
   the path.

## Follow-up

Treat every later question in the conversation as continued onboarding.

- Apply the same evidence order. Cite `path:line` for every claim and write
  `미확인` when no evidence exists.
- Read only the code the question needs.
- When an answer contradicts the briefing, state the correction explicitly.
- When the user asks for a picture, draw one diagram with the rules above:
  Sequence for request or call flows, Dependency graph for module relations, ER
  for data models. Save it as `.onboard/<slug>.html`. When a question is about
  a flow and no picture was requested, offer one in a single line instead of
  drawing it.
- End each answer with one next question tied to evidence found while
  answering.
