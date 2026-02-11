# gnoyes-skills

gnoyes의 개인용 유틸리티 스킬 플러그인 for Claude Code

## 스킬 목록

### GitHub

| 스킬 | 명령어 | 설명 |
|------|--------|------|
| **Smart Commit** | `/gnoyes:smart-commit` | 변경사항을 논리적 단위로 분리하여 커밋 |
| **Create PR** | `/gnoyes:create-pr` | 변경사항 기반으로 Pull Request 생성 |
| **Review PR** | `/gnoyes:review-pr` | PR 리뷰 코멘트 분석 및 요약 |

### English

| 스킬 | 명령어 | 설명 |
|------|--------|------|
| **YT Subtitle** | `/gnoyes:yt-subtitle` | YouTube 영상 자막 추출 → 마크다운 변환 |

## 설치

### 전역 설치 (모든 프로젝트에서 사용)

```bash
claude plugin install /path/to/gnoyes-skills --scope user
```

### 프로젝트별 설치

```bash
cd your-project
claude plugin install /path/to/gnoyes-skills --scope project
```

### 개발/테스트 모드

```bash
claude --plugin-dir /path/to/gnoyes-skills
```

## 사용법

### GitHub

#### Smart Commit (`/gnoyes:smart-commit`)

변경사항을 분석하여 논리적 단위로 분리 커밋합니다.

```bash
# 기본 사용
/gnoyes:smart-commit

# 새 브랜치 생성 후 커밋
/gnoyes:smart-commit --branch feature/new-feature

# 커밋 후 푸시까지
/gnoyes:smart-commit --push

# 브랜치 생성 + 푸시
/gnoyes:smart-commit -b feature/new-feature -p
```

**자동 트리거 키워드**:
- "커밋해줘", "변경사항 정리", "커밋 분리", "논리적으로 커밋"

#### Create PR (`/gnoyes:create-pr`)

현재 브랜치의 변경사항을 기반으로 PR을 생성합니다.

```bash
# 기본 사용 (베이스 브랜치 자동 감지)
/gnoyes:create-pr

# 베이스 브랜치 지정
/gnoyes:create-pr --base develop

# Draft PR로 생성
/gnoyes:create-pr --draft

# 제목 직접 지정
/gnoyes:create-pr --title "feat: 새로운 기능 추가"
```

**자동 트리거 키워드**:
- "PR 만들어", "풀리퀘 생성", "PR 올려", "리뷰 요청"

#### Review PR (`/gnoyes:review-pr`)

PR에 달린 리뷰 코멘트를 분석하고 요약합니다.

```bash
# 현재 브랜치의 PR 분석
/gnoyes:review-pr

# 특정 PR 번호 지정
/gnoyes:review-pr 123
```

**자동 트리거 키워드**:
- "리뷰 확인", "코멘트 요약", "PR 리뷰 정리", "피드백 확인"

### English

#### YT Subtitle (`/gnoyes:yt-subtitle`)

YouTube 영상의 자막을 추출하여 YAML frontmatter가 포함된 마크다운 파일로 저장합니다.

```bash
# 영어 자막 추출
/gnoyes:yt-subtitle https://www.youtube.com/watch?v=dQw4w9WgXcQ en

# 한국어 자막 추출
/gnoyes:yt-subtitle https://youtu.be/dQw4w9WgXcQ ko

# 언어 선택 (대화형)
/gnoyes:yt-subtitle https://www.youtube.com/watch?v=dQw4w9WgXcQ
```

**자동 트리거 키워드**:
- "유튜브 자막", "자막 추출", "YouTube 자막", "subtitle extract"

**사전 요구사항**:
- `yt-dlp` (`brew install yt-dlp`)
- `ffmpeg` 권장 (`brew install ffmpeg`)

## 코멘트 분류 체계

Review PR 스킬은 리뷰 코멘트를 다음과 같이 분류합니다:

| 카테고리 | 아이콘 | 설명 | 액션 |
|----------|--------|------|------|
| MUST_FIX | 🔴 | 반드시 수정 필요 | 필수 |
| SHOULD_FIX | 🟡 | 수정 권장 | 권장 |
| SUGGESTION | 🔵 | 제안/개선 아이디어 | 선택 |
| QUESTION | ❓ | 질문/확인 요청 | 답변 필요 |
| NITPICK | 📝 | 사소한 스타일 이슈 | 선택 |
| PRAISE | 👍 | 칭찬/긍정적 피드백 | - |

## 요구사항

- [Claude Code](https://claude.com/claude-code) CLI
- [GitHub CLI](https://cli.github.com/) (`gh`)
  ```bash
  brew install gh
  gh auth login
  ```

## 디렉토리 구조

```
gnoyes-skills/
├── .claude-plugin/
│   └── plugin.json          # 플러그인 메타데이터
├── skills/
│   ├── smart-commit/
│   │   └── SKILL.md         # Smart Commit 스킬
│   ├── create-pr/
│   │   └── SKILL.md         # Create PR 스킬
│   ├── review-pr/
│   │   └── SKILL.md         # Review PR 스킬
│   └── yt-subtitle/
│       └── SKILL.md         # YT Subtitle 스킬
├── .gitignore
├── LICENSE
└── README.md
```

## 안전 규칙

모든 스킬은 다음 안전 규칙을 따릅니다:

- ❌ `git push --force` 사용 금지
- ❌ 이미 푸시된 커밋 수정 금지
- ✅ 실행 전 항상 사용자 확인
- ✅ 민감 파일 (.env, credentials 등) 감지 시 경고

## 라이선스

MIT License

## 작성자

gnoyes
