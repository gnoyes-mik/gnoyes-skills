---
description: 현재 브랜치의 변경사항을 기반으로 Pull Request를 생성합니다
---

# Create PR

현재 브랜치의 커밋 이력을 분석하여 GitHub Pull Request를 생성합니다.

## 사용법

```
/gnoyes:create-pr [base-branch]
```

- `base-branch`: PR 대상 브랜치 (기본값: develop 또는 main)

## 워크플로우

1. 현재 브랜치와 base 브랜치 간의 차이 분석
2. 커밋 이력에서 PR 제목과 본문 자동 생성
3. `gh pr create` 명령으로 PR 생성
4. PR URL 반환

## PR 본문 형식

```markdown
## Summary
<변경사항 요약 - 1-3개 bullet points>

## Test plan
<테스트 계획 체크리스트>

🤖 Generated with Claude Code
```

## gh CLI 필수

이 명령은 GitHub CLI (`gh`)가 필요합니다:

```bash
gh auth status
```
