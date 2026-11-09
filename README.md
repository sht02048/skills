# Skills

Codex와 Claude Code에서 함께 쓰는 개인 에이전트 스킬 모음이에요. 커밋부터
PR, 리뷰 진단, 티켓 병렬 구현까지 한국어 개발 흐름에 맞춰 다듬었고,
어느 저장소에서든 한 줄로 설치해 바로 쓸 수 있어요.

## 빠른 시작

Node.js만 있으면 돼요. 설치하면 새 세션에서 바로 쓸 수 있고, 최신
버전으로 올릴 때도 같은 명령을 다시 실행하면 돼요.

```bash
# Codex
npx skills add sht02048/skills --global --agent codex --skill '*' --yes

# Claude Code
npx skills add sht02048/skills --global --agent claude-code --skill '*' --yes
```

설치가 끝나면 `$commit`(Codex) 또는 `/commit`(Claude Code)처럼 불러요.
다른 설치 방식은 [설치](#설치)에서 고를 수 있어요.

## 스킬

### Git 흐름

| 스킬 | 하는 일 | 이렇게 말하면 |
| --- | --- | --- |
| `commit` | staged 변경을 먼저 존중하고, 없으면 관심사별로 나눠 한국어 gitmoji 커밋을 만들어요. `git add .`이나 `--no-verify`는 쓰지 않아요. | "커밋", "나눠서 커밋" |
| `pr` | 베이스 브랜치를 확인하고 PR 템플릿을 지켜 GitHub PR을 올린 뒤, 프로젝트가 고른 검증 게이트(React Doctor 같은 명령, 독립 에이전트 리뷰, CI, 자동 Codex 리뷰)로 head SHA 기준 판정을 내려요. 게이트는 말로 추가하거나 바꿀 수 있어요. | "PR 올려줘", "PR 게이트 추가" |
| `pr-review-diagnosis` | 현재 브랜치 PR의 리뷰 스레드를 분류하고, 최신 커밋에서 어떤 체크가 왜 실패했는지 알려줘요. | "PR 리뷰 봐줘", "CI 왜 실패했어" |

### 전달

| 스킬 | 하는 일 | 이렇게 말하면 |
| --- | --- | --- |
| `orca-orchestration` | 티켓마다 Orca 워커 워크트리를 만들어 구현을 맡기고, React Doctor와 자동 Codex 리뷰를 통과한 PR만 머지한 뒤 워크트리를 깨끗하게 정리해요. 코디네이터는 코드를 건드리지 않아요. | "오르카로 처리", "티켓 병렬로 구현" |

`grill-me → to-spec → to-tickets → implement-spec` 흐름의 마지막 단계에서
티켓을 병렬로 구현할 때 주로 쓰지만, 출처와 완료 기준이 분명한 티켓이면
무엇이든 돼요. Orca가 실행 중이어야 하고, CLI 조작은 Orca가 제공하는
`orchestration` 스킬에 맡겨요. 머지한 워크트리의 사본과 빌드 산출물이
쌓여 디스크가 가득 찼던 적이 있어서, 다음 작업에 넘길 것을 넘기고
워크트리까지 정리해야 끝나요.

### 코드베이스 이해

| 스킬 | 하는 일 | 이렇게 말하면 |
| --- | --- | --- |
| `onboard` | 낯선 저장소의 목적, 스택, 명령, 구조, 관례, 함정을 근거와 함께 설명하고 이어지는 질문에 답해요. 직접 부를 때만 움직여요. | "온보딩", "이 레포 설명해줘" |
| `dry-audit` | 호출·의존 그래프로 중복되거나 갈라진 로직을 찾아 P0~P3로 우선순위를 매겨요. 코드는 고치지 않아요. | "중복 로직 찾아줘", "DRY 검사" |
| `llm-wiki-update` | 소스, 설정, 테스트, Git 기록을 근거로 저장소의 `llm-wiki`를 기존 관례대로 최신 상태에 맞춰요. | "위키 갱신" |

### 글쓰기

| 스킬 | 하는 일 | 이렇게 말하면 |
| --- | --- | --- |
| `ux-writing` | 토스 UX 라이팅 원칙으로 한국어 UI 문구를 쓰거나 고치고, 적용한 규칙과 함께 Before/After 표로 보여줘요. | "문구 다듬어줘", "토스 스타일로" |
| `skill-refine` | 기존 스킬의 모호한 규칙, 빠진 근거, 약한 트리거를 찾아 의도를 지키면서 엄격하게 다시 써요. | "스킬 개선", "스킬 검사" |

## 설치

### Codex

사용자 범위에 설치하면 모든 프로젝트에서 쓸 수 있어요.

```bash
npx skills add sht02048/skills --global --agent codex --skill '*' --yes
```

한 프로젝트에서만 쓰려면 그 프로젝트 루트에서 `--global`을 빼고
실행하면 돼요. 스킬은 `.agents/skills`에 들어가요.

### Claude Code

두 방식 중 하나를 고르면 돼요.

**플러그인.** 이 저장소를 마켓플레이스로 추가하고 설치해요. 스킬은
`/skills:commit`처럼 플러그인 이름을 붙여 불러요.

```text
/plugin marketplace add sht02048/skills
/plugin install skills@sht02048-skills
```

최신 버전으로 올리려면 다음 한 줄을 실행해요.

```text
/plugin marketplace update sht02048-skills
```

**스킬만 설치.** `/commit`처럼 접두사 없이 부르려면 `~/.claude/skills`에
직접 넣어요. 프로젝트 범위로 쓰려면 `--global`을 빼고 프로젝트 루트에서
실행하면 돼요.

```bash
npx skills add sht02048/skills --global --agent claude-code --skill '*' --yes
```

## 구조

스킬 하나가 폴더 하나예요.

```text
skills/<name>/
├── SKILL.md            지침. frontmatter의 description이 트리거 조건
├── agents/openai.yaml  Codex UI에 보이는 이름과 기본 프롬프트
└── references/         필요할 때만 읽는 긴 자료 (일부 스킬)
```

"언제 쓰는지"는 전부 `SKILL.md`의 description에 적어요. 에이전트가 스킬을
쓸지 판단할 때는 description만 보이기 때문이에요. 본문은 에이전트가
스킬을 고른 뒤에 읽고, references는 본문이 가리킬 때만 읽어요.
