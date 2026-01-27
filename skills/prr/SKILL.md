---
name: prr
description: |
  PR에 달린 리뷰 코멘트를 분석하고 요약합니다. 변경 필요 여부를 판단합니다.
  트리거: "리뷰 확인", "코멘트 요약", "PR 리뷰 정리", "리뷰 코멘트",
  "review summary", "피드백 확인", "리뷰어 의견", "PR 피드백"
disable-model-invocation: false
allowed-tools: Read, Grep, Bash(git:*), Bash(gh:*)
argument-hint: "[PR번호]"
---

# PR Review Summary - 리뷰 코멘트 분석 도구

PR에 달린 리뷰 코멘트를 수집, 분류, 요약하고 대응 방안을 제안합니다.

## 옵션

| 옵션 | 설명 |
|------|------|
| `<PR번호>` | 특정 PR 번호 지정 |
| (없음) | 현재 브랜치의 PR 자동 감지 |

## 실행 워크플로우

### Step 1: PR 식별

PR 번호가 없으면 현재 브랜치의 PR을 찾습니다:

```bash
gh pr view --json number,title,state,author,reviewDecision
```

### Step 2: 리뷰 코멘트 수집

```bash
# PR 리뷰 전체 정보
gh pr view <number> --json reviews,comments

# 상세 코멘트 (파일별 인라인 코멘트)
gh api repos/{owner}/{repo}/pulls/<number>/comments

# 일반 코멘트
gh api repos/{owner}/{repo}/issues/<number>/comments
```

### Step 3: 코멘트 분류

각 코멘트를 다음 카테고리로 분류합니다:

| 카테고리 | 아이콘 | 설명 | 액션 필요 |
|----------|--------|------|----------|
| **MUST_FIX** | 🔴 | 반드시 수정 필요 | ✅ 필수 |
| **SHOULD_FIX** | 🟡 | 수정 권장 | ⚠️ 권장 |
| **SUGGESTION** | 🔵 | 제안/개선 아이디어 | 선택적 |
| **QUESTION** | ❓ | 질문/확인 요청 | 답변 필요 |
| **NITPICK** | 📝 | 사소한 스타일 이슈 | 선택적 |
| **PRAISE** | 👍 | 칭찬/긍정적 피드백 | 없음 |

### 분류 기준

**MUST_FIX 키워드**:
- "must", "반드시", "필수", "버그", "오류", "잘못"
- "보안", "취약점", "위험", "critical"
- Request Changes 상태의 리뷰

**SHOULD_FIX 키워드**:
- "should", "권장", "좋겠", "하면 좋"
- "성능", "효율", "개선"
- "recommend", "suggest strongly"

**SUGGESTION 키워드**:
- "제안", "어떨까", "고려", "대안"
- "could", "might", "maybe"
- "idea", "alternative"

**QUESTION 키워드**:
- "?", "왜", "어떻게", "의도"
- "확인", "질문", "궁금"
- "why", "how", "what"

**NITPICK 키워드**:
- "nitpick", "nit", "사소"
- "포맷", "띄어쓰기", "오타"
- "minor", "trivial"

**PRAISE 키워드**:
- "좋", "훌륭", "깔끔", "nice", "great", "good"
- "👍", "LGTM", "잘했", "멋지"

### Step 4: 요약 리포트 생성

**출력 형식**:

```markdown
# 📋 PR Review Summary

**PR**: #123 - feat: 사용자 인증 구현
**Author**: @gnoyes
**리뷰어**: @reviewer1, @reviewer2
**상태**: Changes Requested
**총 코멘트**: 12개

---

## 📊 요약 통계

| 카테고리 | 개수 | 상태 |
|----------|------|------|
| 🔴 MUST_FIX | 3 | ⏳ 수정 필요 |
| 🟡 SHOULD_FIX | 2 | ⏳ 권장 |
| 🔵 SUGGESTION | 3 | 선택 |
| ❓ QUESTION | 2 | 답변 필요 |
| 📝 NITPICK | 1 | 선택 |
| 👍 PRAISE | 1 | - |

---

## 🔴 필수 수정 (MUST_FIX) - 3개

### 1. SQL Injection 취약점
- **파일**: `UserRepository.java:45`
- **리뷰어**: @reviewer1
- **내용**: 동적 쿼리에서 파라미터 바인딩 없이 직접 문자열 결합 사용
- **제안 해결책**: PreparedStatement 또는 JOOQ의 파라미터 바인딩 사용

### 2. Null 체크 누락
- **파일**: `AuthService.java:78`
- **리뷰어**: @reviewer2
- **내용**: `user`가 null일 수 있는데 체크 없이 사용
- **제안 해결책**: `@NonNull` 검증 또는 Optional 처리

### 3. 테스트 커버리지 부족
- **파일**: `AuthServiceTest.java`
- **리뷰어**: @reviewer1
- **내용**: 예외 케이스 테스트가 없음
- **제안 해결책**: 인증 실패 시나리오 테스트 추가

---

## 🟡 권장 수정 (SHOULD_FIX) - 2개

### 1. 메서드 분리 권장
- **파일**: `UserController.java:120`
- **리뷰어**: @reviewer1
- **내용**: 메서드가 너무 길어 가독성 저하
- **제안 해결책**: private 메서드로 분리

### 2. 상수 추출
- **파일**: `JwtUtil.java:35`
- **리뷰어**: @reviewer2
- **내용**: 매직 넘버 사용
- **제안 해결책**: 상수로 추출하여 의미 명확히

---

## 🔵 제안사항 (SUGGESTION) - 3개

1. **로깅 추가** (`AuthService.java:50`) - @reviewer1
   - 인증 실패 시 로그 기록 권장

2. **캐싱 고려** (`UserRepository.java:30`) - @reviewer2
   - 자주 조회되는 사용자 정보 캐싱 제안

3. **Builder 패턴** (`UserDto.java`) - @reviewer1
   - 필드가 많으므로 Builder 패턴 제안

---

## ❓ 질문/확인 필요 (QUESTION) - 2개

### 1. 토큰 만료 시간 의도 확인
- **파일**: `JwtUtil.java:20`
- **리뷰어**: @reviewer2
- **질문**: "토큰 만료 시간이 1시간인 이유가 있나요?"
- **답변 제안**: 보안과 UX 균형을 위한 선택. 필요시 설정 가능하게 변경 가능.

### 2. 예외 처리 전략
- **파일**: `AuthService.java:90`
- **리뷰어**: @reviewer1
- **질문**: "모든 예외를 AuthException으로 래핑하는 이유?"
- **답변 제안**: 클라이언트에 일관된 에러 응답 제공 목적.

---

## 📝 Nitpicks - 1개

1. **변수명 개선** (`AuthService.java:25`) - @reviewer2
   - `u` → `user`로 변경 권장

---

## 👍 긍정적 피드백 - 1개

1. @reviewer1: "전체적으로 깔끔한 구조입니다. 특히 JwtUtil 분리가 좋네요! 👍"

---

## 📋 액션 아이템 요약

| 우선순위 | 파일 | 항목 | 예상 시간 |
|----------|------|------|----------|
| 🔴 P0 | UserRepository.java:45 | SQL Injection 수정 | ~10분 |
| 🔴 P0 | AuthService.java:78 | Null 체크 추가 | ~5분 |
| 🔴 P0 | AuthServiceTest.java | 예외 테스트 추가 | ~15분 |
| 🟡 P1 | UserController.java:120 | 메서드 분리 | ~15분 |
| 🟡 P1 | JwtUtil.java:35 | 상수 추출 | ~5분 |

**총 예상 작업 시간**: 약 50분

---

## 🚀 다음 단계

위 항목들을 수정 후:
1. `/gnoyes:smart-commit` 으로 변경사항 커밋
2. `gh pr review --comment -b "리뷰 반영 완료"` 로 리뷰어에게 알림
```

### Step 5: 대응 방안 제안

각 MUST_FIX, SHOULD_FIX 항목에 대해:
- 구체적인 코드 수정 방안 제시
- 관련 프로젝트 규칙 참조 (CLAUDE.md가 있다면)
- 수정에 필요한 예상 시간

### Step 6: 후속 액션 제안

분석 완료 후 다음 옵션을 제시:

```
다음 중 원하는 작업을 선택해주세요:

1. 🔧 필수 수정 사항(MUST_FIX) 자동 수정 시작
2. 💬 질문(QUESTION)에 대한 답변 코멘트 작성
3. ✅ 전체 수정 후 리뷰 재요청
4. 📝 특정 항목만 수정
```

## 리뷰 상태별 처리

| PR 상태 | 처리 |
|---------|------|
| `APPROVED` | 긍정 피드백 요약, 머지 준비 확인 |
| `CHANGES_REQUESTED` | 필수 수정 사항 강조 |
| `COMMENTED` | 전체 코멘트 분석 |
| `PENDING` | 리뷰 대기 중 알림 |

## 예시

### 입력
```
/gnoyes:prr 123
```

또는 현재 브랜치에서:
```
/gnoyes:prr
```

### 자동 트리거 예시

사용자: "리뷰 코멘트 확인해줘"
→ 자동으로 `/gnoyes:prr` 스킬 실행

## 안전 규칙

- ✅ 읽기 전용 작업만 수행 (코멘트 수집 및 분석)
- ✅ 코드 수정은 사용자 확인 후 별도 진행
- ✅ 리뷰어 의견 존중, 일방적 무시 방지
- ✅ 민감한 코멘트 (개인 비판 등)는 중립적으로 요약

## gh CLI 필수

이 스킬은 GitHub CLI (`gh`)가 필요합니다:

```bash
gh --version
gh auth status
```

## 관련 스킬

- `/gnoyes:smart-commit` - 수정 후 커밋
- `/gnoyes:pr` - 새 PR 생성
