# Codex Skills

다른 저장소에서도 사용할 수 있는 개인 Codex 스킬 모음입니다.

## 설치

Node.js가 필요합니다. 설치 후 새 Codex 세션에서 사용할 수 있습니다.

### 사용자 범위

모든 프로젝트에서 사용하려면 전체 스킬을 Codex 사용자 범위에
설치합니다.

```bash
npx skills add sht02048/skills --global --agent codex --skill '*' --yes
```

설치된 스킬을 최신 버전으로 갱신하려면 다음 명령을 실행합니다.

```bash
npx skills update --global
```

### 프로젝트 범위

현재 프로젝트에서만 사용하려면 대상 프로젝트 루트에서 다음 명령을
실행합니다. 스킬은 `.agents/skills`에 설치됩니다.

```bash
npx skills add sht02048/skills --agent codex --skill '*' --yes
```

프로젝트에 설치된 스킬을 최신 버전으로 갱신하려면 다음 명령을
실행합니다.

```bash
npx skills update --project
```

## 스킬

- `commit`: staged-first 한국어 Git 커밋
- `pr`: 베이스 브랜치를 확인하고 GitHub PR 생성
- `pr-review-diagnosis`: 현재 브랜치 PR 리뷰 진단
- `llm-wiki-update`: 저장소 근거를 반영해 기존 `llm-wiki` 갱신
