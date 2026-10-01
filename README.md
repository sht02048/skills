# Skills

다른 저장소에서도 사용할 수 있는 개인 Codex · Claude Code 스킬 모음입니다.

## Codex 설치

Node.js가 필요합니다. 설치 후 새 Codex 세션에서 사용할 수 있습니다.

### 사용자 범위

모든 프로젝트에서 사용하려면 전체 스킬을 Codex 사용자 범위에
설치합니다. 최신 버전으로 갱신할 때도 같은 명령을 다시 실행합니다.

```bash
npx skills add sht02048/skills --global --agent codex --skill '*' --yes
```

### 프로젝트 범위

현재 프로젝트에서만 사용하려면 대상 프로젝트 루트에서 다음 명령을
실행합니다. 스킬은 `.agents/skills`에 설치됩니다. 최신 버전으로
갱신할 때도 같은 명령을 다시 실행합니다.

```bash
npx skills add sht02048/skills --agent codex --skill '*' --yes
```

## Claude Code 설치

### 플러그인

Claude Code에서 이 저장소를 마켓플레이스로 추가한 뒤 플러그인을
설치합니다. 스킬은 `/skills:commit`처럼 플러그인 이름이 붙은 형태로
호출합니다.

```text
/plugin marketplace add sht02048/skills
/plugin install skills@sht02048-skills
```

최신 버전으로 갱신하려면 다음 명령을 실행합니다.

```text
/plugin marketplace update sht02048-skills
```

### 스킬만 설치

`/commit`처럼 접두사 없이 호출하려면 스킬을 `~/.claude/skills`에 직접
설치합니다. Node.js가 필요하며, 갱신할 때도 같은 명령을 다시
실행합니다. 프로젝트 범위로 설치하려면 `--global`을 빼고 대상 프로젝트
루트에서 실행합니다.

```bash
npx skills add sht02048/skills --global --agent claude-code --skill '*' --yes
```

## 스킬

- `commit`: staged-first 한국어 Git 커밋
- `dry-audit`: 그래프 근거로 중복 책임을 찾아 P0-P3로 우선순위화
- `orca-orchestration`: Orca worktree 기반 구현과 PR 병합 게이트 조율
- `pr`: 베이스 브랜치를 확인하고 GitHub PR 생성
- `pr-review-diagnosis`: 현재 브랜치 PR 리뷰 및 액션 실패 진단
- `llm-wiki-update`: 저장소 근거를 반영해 기존 `llm-wiki` 갱신
