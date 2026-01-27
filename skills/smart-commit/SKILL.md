---
name: smart-commit
description: 변경사항을 논리적 단위로 분리하여 커밋합니다. 커밋해줘, 변경사항 정리, 커밋 분리, smart commit
---

# Smart Commit - 논리적 커밋 분리 도구

변경사항을 분석하여 논리적 개발 단위로 분리 커밋합니다.

## 옵션

| 옵션 | 단축 | 설명 |
|------|------|------|
| `--push` | `-p` | 커밋 후 자동 푸시 |
| `--branch <name>` | `-b <name>` | 새 브랜치 생성 후 커밋 |
| (없음) | | 현재 브랜치에 커밋만 |

## 실행 워크플로우

### Step 1: 현재 상태 분석

다음 명령어로 변경 상태를 파악합니다:

```bash
git status
git diff --stat
git diff --name-only
```

변경된 파일 목록과 변경 내용을 분석합니다.

### Step 2: 논리적 단위 분류

변경사항을 다음 기준으로 분류합니다:

| 분류 기준 | 예시 |
|----------|------|
| **기능 단위** | 같은 기능에 속하는 파일들 |
| **레이어 단위** | Entity, Repository, Service, Controller |
| **도메인 단위** | User 관련, Order 관련 |
| **타입 단위** | 프로덕션 코드, 테스트 코드, 설정 파일 |

### Step 3: 커밋 계획 제안

각 논리적 단위에 대해 다음을 결정합니다:
- 포함될 파일 목록
- 제안 커밋 메시지 (Conventional Commits 형식)
- 커밋 순서 (의존성 고려)

**커밋 메시지 형식**:
```
<type>(<scope>): <subject>

<body>
```

| Type | 용도 |
|------|------|
| `feat` | 새 기능 |
| `fix` | 버그 수정 |
| `refactor` | 리팩토링 |
| `test` | 테스트 추가/수정 |
| `docs` | 문서 |
| `chore` | 빌드, 설정 |
| `style` | 코드 포맷팅 |
| `perf` | 성능 개선 |

### Step 4: 사용자 확인

커밋 계획을 보여주고 **반드시 사용자 승인**을 받습니다:

> "다음과 같이 N개의 커밋으로 분리하겠습니다. 진행할까요?"

**확인 사항**:
- 각 커밋에 포함될 파일 목록
- 제안된 커밋 메시지
- 커밋 순서

### Step 5: 커밋 실행

승인 후 각 단위별로 실행:

```bash
git add <files>
git commit -m "<message>"
```

### Step 6: 선택적 푸시

`--push` 옵션이 있거나 사용자가 요청한 경우:

```bash
git push -u origin <branch>
```

## 브랜치 생성 시

`--branch` 옵션이 있는 경우:

1. **브랜치명 검증** (kebab-case 권장)
2. **브랜치 생성**: `git checkout -b <branch>`
3. 이후 커밋 진행

**브랜치명 제안 규칙**:
- `feature/<scope>-<description>`
- `fix/<scope>-<description>`
- `refactor/<scope>-<description>`

## 예시 시나리오

### 입력
```
/gnoyes:smart-commit --branch feature/user-auth --push
```

### 분석 결과
```
변경 파일 8개 감지:
- UserEntity.java, UserRepository.java (User 도메인)
- AuthService.java, JwtUtil.java (인증 로직)
- UserController.java (API 엔드포인트)
- UserServiceTest.java, AuthServiceTest.java (테스트)
- application.yml (설정)
```

### 커밋 계획
```
1. feat(user): Add User entity and repository
   - UserEntity.java
   - UserRepository.java

2. feat(auth): Implement JWT authentication service
   - AuthService.java
   - JwtUtil.java

3. feat(api): Add user authentication endpoints
   - UserController.java

4. test(auth): Add unit tests for authentication
   - UserServiceTest.java
   - AuthServiceTest.java

5. chore(config): Add JWT configuration
   - application.yml
```

## 민감 파일 감지

다음 파일이 변경사항에 포함된 경우 **경고**합니다:

| 파일 패턴 | 위험도 |
|----------|--------|
| `.env*` | 🔴 높음 |
| `*credentials*` | 🔴 높음 |
| `*secret*` | 🔴 높음 |
| `*.pem`, `*.key` | 🔴 높음 |
| `application-prod*.yml` | 🟡 중간 |

## 안전 규칙

- ❌ `--force` 옵션 절대 사용 금지
- ❌ 이미 푸시된 커밋 수정 금지
- ❌ `--amend` 사용 금지 (새 커밋 생성)
- ✅ 커밋 전 항상 사용자 확인
- ✅ 민감 파일 감지 시 경고
- ✅ `.gitignore` 패턴 존중

## 기존 커밋 컨벤션 분석

프로젝트의 기존 커밋 메시지를 분석하여 스타일을 맞춥니다:

```bash
git log --oneline -20
```

분석 항목:
- 메시지 언어 (한국어/영어)
- Type 사용 여부
- Scope 사용 패턴
- Subject 길이
