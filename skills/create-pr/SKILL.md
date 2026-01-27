---
name: create-pr
description: 현재 브랜치의 변경사항을 기반으로 Pull Request를 생성합니다. PR 만들어, 풀리퀘 생성, create pr
---

# PR Create - Pull Request 생성 도구

현재 브랜치의 변경사항 또는 커밋 이력을 분석하여 PR을 생성합니다.

## 옵션

| 옵션 | 단축 | 설명 |
|------|------|------|
| `--draft` | `-d` | Draft PR로 생성 |
| `--base <branch>` | `-b` | 베이스 브랜치 지정 (기본: develop 또는 main) |
| `--title <title>` | `-t` | PR 제목 직접 지정 |
| (없음) | | 자동 분석 후 PR 생성 |

## 실행 워크플로우

### Step 1: 현재 상태 확인

```bash
git branch --show-current
git log origin/develop..HEAD --oneline
git diff origin/develop...HEAD --stat
```

### Step 2: 베이스 브랜치 결정

우선순위:
1. `--base` 옵션으로 지정된 브랜치
2. 프로젝트 기본 브랜치 감지 (develop > main > master)

```bash
# 기본 브랜치 감지
git remote show origin | grep 'HEAD branch'
```

### Step 3: 변경사항 분석

**분석 대상**:
- 커밋 메시지들
- 변경된 파일 목록
- 코드 diff 요약

**추출 정보**:
- 주요 변경 내용
- 변경 타입 (feature, fix, refactor 등)
- 영향 범위 (어떤 모듈/도메인)

### Step 4: PR 제목 생성

**형식**: `<type>: <간결한 설명>`

| 브랜치 패턴 | 제목 예시 |
|------------|----------|
| `feature/user-auth` | `feat: 사용자 인증 기능 구현` |
| `fix/login-error` | `fix: 로그인 오류 수정` |
| `refactor/api-layer` | `refactor: API 레이어 리팩토링` |

브랜치명에서 타입과 설명을 추출합니다.

### Step 5: PR 본문 생성

**템플릿**:

```markdown
## Summary
<1-3개의 핵심 변경사항 bullet points>

## Changes
<변경된 주요 파일/모듈 설명>

### 주요 변경 파일
- `path/to/file1.java` - 설명
- `path/to/file2.java` - 설명

## Test Plan
- [ ] 단위 테스트 통과
- [ ] 통합 테스트 통과
- [ ] 수동 테스트 완료

## Checklist
- [ ] 코드 스타일 가이드 준수
- [ ] 문서 업데이트 (필요시)
- [ ] Breaking changes 없음

## Related Issues
<관련 이슈 링크 - 있는 경우>

---
🤖 Generated with Claude Code
```

### Step 6: 사용자 확인

PR 제목과 본문을 보여주고 **반드시 확인**:

> "다음 내용으로 PR을 생성합니다. 수정이 필요하면 말씀해주세요."

사용자가 수정을 요청하면 반영합니다.

### Step 7: 푸시 확인 및 실행

로컬 커밋이 푸시되지 않았다면:

```bash
git push -u origin <current-branch>
```

### Step 8: PR 생성

```bash
gh pr create --title "<title>" --body "<body>" [--draft] --base <base>
```

### Step 9: 결과 출력

```
✅ PR 생성 완료!
🔗 https://github.com/owner/repo/pull/123

📋 요약:
- 제목: feat: 사용자 인증 기능 구현
- 베이스: develop
- 상태: Ready for Review (또는 Draft)
```

## 프로젝트별 PR 템플릿

### Java/Spring 프로젝트 (revn-backend 등)

PR 본문에 추가 섹션:

```markdown
## Architecture Compliance
- [ ] 생성자 주입만 사용 (필드 주입 금지)
- [ ] JSpecify 애노테이션 사용 (@NonNull, @Nullable)
- [ ] DTO는 Record 타입으로 구현
- [ ] JPA Entity에 @Column 명시

## Quality Checks
- [ ] `./gradlew spotlessApply` 실행
- [ ] `./gradlew test` 통과
- [ ] 아키텍처 테스트 통과
```

### Frontend 프로젝트

```markdown
## Browser Testing
- [ ] Chrome
- [ ] Firefox
- [ ] Safari

## Responsive
- [ ] Desktop (1920px)
- [ ] Tablet (768px)
- [ ] Mobile (375px)
```

## 예시

### 입력
```
/gnoyes:create-pr --base develop
```

### 출력 미리보기
```markdown
## PR Preview

**Title**: feat: CMS 회원 테이블 설계

**Base**: develop ← feature/cms-member-table-design

**Body**:
## Summary
- CMS 회원 관리를 위한 테이블 스키마 설계
- Flyway 마이그레이션 스크립트 추가
- 관련 Entity 및 Repository 구현

## Changes
### 주요 변경 파일
- `revn-core/src/main/resources/db/migration/V20260127_01__xxx.sql`
- `revn-core/src/main/java/.../entity/CmsMemberEntity.java`
- `revn-core/src/main/java/.../repository/CmsMemberRepository.java`

## Test Plan
- [x] 아키텍처 테스트 통과
- [x] Spotless 포맷팅 적용
- [ ] 통합 테스트 작성

---
🤖 Generated with Claude Code

이대로 PR을 생성할까요? (수정이 필요하면 말씀해주세요)
```

## 안전 규칙

- ✅ PR 생성 전 항상 내용 확인
- ✅ 민감 정보 포함 여부 체크 (API 키, 비밀번호 등)
- ✅ 베이스 브랜치 확인 (실수로 main에 직접 PR 방지)
- ✅ 커밋이 푸시되었는지 확인
- ⚠️ Force push가 필요한 상황이면 경고

## gh CLI 필수

이 스킬은 GitHub CLI (`gh`)가 필요합니다:

```bash
# 설치 확인
gh --version

# 로그인 확인
gh auth status
```

설치되지 않았다면 안내합니다:
```bash
brew install gh
gh auth login
```
