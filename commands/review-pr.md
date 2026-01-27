---
description: PR에 달린 리뷰 코멘트를 분석하고 요약합니다
---

# Review PR

PR에 달린 리뷰 코멘트를 수집, 분류, 요약하고 대응 방안을 제안합니다.

## 사용법

```
/gnoyes:review-pr [PR번호]
```

- `PR번호`: 특정 PR 번호 (없으면 현재 브랜치의 PR 자동 감지)

## 코멘트 분류

| 카테고리 | 아이콘 | 설명 |
|----------|--------|------|
| MUST_FIX | 🔴 | 반드시 수정 필요 |
| SHOULD_FIX | 🟡 | 수정 권장 |
| SUGGESTION | 🔵 | 제안/개선 아이디어 |
| QUESTION | ❓ | 질문/확인 요청 |
| NITPICK | 📝 | 사소한 스타일 이슈 |
| PRAISE | 👍 | 칭찬/긍정적 피드백 |

## 워크플로우

1. PR 정보 수집 (`gh pr view`)
2. 리뷰 코멘트 수집 (`gh api`)
3. 코멘트 분류 및 우선순위 지정
4. 요약 리포트 생성
5. 대응 방안 제안

## gh CLI 필수

이 명령은 GitHub CLI (`gh`)가 필요합니다:

```bash
gh auth status
```
