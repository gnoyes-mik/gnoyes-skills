---
name: linkedin-knowledge-organizer
description: LinkedIn 활동(퍼온글/좋아요/코멘트/원글)을 Obsidian 지식 저장소에 카테고리화하여 중복 없이 정리합니다. 본인 프로필은 기본값이며 인자로 타인 프로필 URL을 넘기면 해당 사용자 활동도 정리 가능. 링크드인 지식정리, 퍼온글 정리, Obsidian knowledge base, LinkedIn organize
---

# LinkedIn Knowledge Organizer

LinkedIn activity(본인 또는 지정한 타인 프로필)의 원글·공유(Repost)·좋아요·코멘트를 수집하여, 본문과 포함 링크의 내용을 함께 분석한 뒤 Obsidian 지식 저장소에 **중복 없이 카테고리화**하여 저장·갱신합니다. 노트는 Obsidian `[[위키링크]]`로 서로 연결되어 지식 그래프를 구성합니다.

## 인자

- `$ARGUMENTS`: (선택) 타인 LinkedIn 프로필 URL (예: `https://www.linkedin.com/in/someone/`). 미지정 시 본인 프로필(`linkedin.com/in/me`).

## 사전 요구사항

- Chrome 브라우저에서 LinkedIn 로그인 상태
- `claude-in-chrome` MCP 설치 및 연결
- `~/.claude/settings.json`에 vault 경로 설정:
  ```json
  {
    "gnoyes": {
      "linkedin_organizer": {
        "vault_path": "/absolute/path/to/obsidian-vault",
        "subfolder": "Sources/LinkedIn"
      }
    }
  }
  ```
- WebFetch 도구 사용 가능

## 실행 워크플로우

### Step 1: 환경 확인 및 모드 결정

1. `mcp__claude-in-chrome__tabs_context_mcp`로 Chrome MCP 연결 확인
   - 실패 시 → **Step 1-B 폴백 모드**
2. `~/.claude/settings.json`을 Read로 로드
   - `gnoyes.linkedin_organizer.vault_path` 미설정 시: 설정 방법 안내 후 중단
   - `subfolder` 기본값: `Sources/LinkedIn`
3. `vault_path` 디렉토리 존재 여부 및 쓰기 가능 여부 확인
4. **모드 결정**:
   - `$ARGUMENTS` 없음 → **본인 모드**
     - 대상 URL: `https://www.linkedin.com/in/me`
     - 저장 경로: `{vault_path}/{subfolder}/me/`
     - `profile_owner = "me"`
   - `$ARGUMENTS`가 `https://www.linkedin.com/in/{username}/?` 패턴에 매칭 → **타인 모드**
     - username 추출, `profile_owner = username`
     - 저장 경로: `{vault_path}/{subfolder}/{username}/`
     - 사용자에게 대상 프로필을 명시하고 **명시적 진행 확인**
   - 그 외 형식 → 사용법 안내 후 중단
5. `vault_path`가 `.env`, `credentials`, `.ssh` 등 민감 경로를 포함하면 경고하고 확인

### Step 1-B: 폴백 모드 (Chrome MCP 미설치)

> "Chrome MCP가 연결되지 않았습니다. 수동 모드로 전환합니다.
>
> LinkedIn activity 페이지(본인 또는 대상 프로필의 `/recent-activity/all/`)에서 아래 정보를 붙여넣어 주세요:
> 1. 각 항목의 타입(원글/Repost/Like/Comment)
> 2. 원글 작성자, 본문, 포함 링크 URL
> 3. (해당 시) 본인 코멘트
> 4. 게시 날짜 및 permalink(가능한 경우)"

수동 입력을 받은 후 Step 3으로 건너뜁니다.

### Step 2: 활동 수집 (Chrome MCP)

대상 프로필의 활동을 수집합니다. **본인 모드**는 `/in/me`, **타인 모드**는 `$ARGUMENTS`의 username을 사용합니다.

**2-1. Activity 페이지 순회:**

1. `mcp__claude-in-chrome__navigate`로 각 탭 이동 후 `mcp__claude-in-chrome__get_page_text`로 텍스트 수집
   - `{profile_url}/recent-activity/all/`
   - `{profile_url}/recent-activity/shares/`
   - **본인 모드 전용**: `.../recent-activity/comments/`, `.../recent-activity/reactions/`
2. 스크롤로 충분한 범위를 확보 (증분 수집용 — 이미 저장된 가장 최근 항목보다 이전까지)
3. 타인 모드에서는 공개 범위에 한함. 비공개/제한 항목은 건너뜀

**2-2. 항목 정규화 (각 activity 당):**

```json
{
  "urn": "urn:li:activity:...",
  "activity_type": "original | repost | like | comment",
  "profile_owner": "me 또는 username",
  "profile_url": "https://www.linkedin.com/in/...",
  "post_author": "원글 작성자 (repost/like/comment 시)",
  "text": "원글 본문",
  "my_comment": "본인이 남긴 코멘트 (해당 시)",
  "urls": ["https://..."],
  "posted_at": "YYYY-MM-DD",
  "permalink": "https://www.linkedin.com/feed/update/..."
}
```

### Step 3: Vault 스캔 (중복 인덱스 + 카테고리 컨텍스트)

1. `{vault_path}/{subfolder}/{profile_owner}/**/*.md` 순회하여 frontmatter 파싱 (Glob + Read)
2. **중복 인덱스 구축**:
   - `linkedin_urn` → 파일 경로 맵
   - 정규화된 URL(utm_*, fbclid, gclid 제거, trailing slash 정규화) → 파일 경로 맵
3. **카테고리 컨텍스트** 수집 (프로필 owner 범위):
   - 기존 카테고리 폴더명 목록
   - 기존 노트들의 `tags`, `category` frontmatter 값 집계
   - `_MOC.md` 파일 목록
4. 프로필 간 오탐을 막기 위해 중복 스캔은 `{profile_owner}` 범위로 한정. 단, 다른 프로필에서 동일 외부 URL이 이미 저장되어 있다면 `related_notes[]`에 교차 `[[...]]` 링크로 기록

### Step 4: 링크 보강 (WebFetch)

각 수집 항목의 `urls[]`에 대해:

1. WebFetch로 페이지 가져오기 (실패·타임아웃은 graceful하게 스킵)
2. 추출: `{ url, title, summary(1-2문단), domain }`
3. 결과를 항목의 `link_context[]`에 저장. 분류 단계 입력으로 사용

### Step 5: 중복 판정 및 카테고리화

**5-1. 중복 판정 (항목별):**

1. `linkedin_urn`이 인덱스에 존재 → **SKIP** (이미 저장됨)
2. 정규화 URL 중 하나라도 인덱스에 존재 → **MERGE**
   - 기존 노트 말미에 `## 추가 맥락 ({collected_at})` 섹션을 append
   - 본인 코멘트 / 새로운 코멘트 / 새 permalink / `source_urls[]` 보강
3. 그 외 → **NEW**

**5-2. 카테고리 결정 (NEW 항목에 대해 LLM 판단):**

입력:
- 원글 본문
- 본인 코멘트 (있을 경우)
- `link_context[]` (제목 + 요약 + 도메인)
- **Vault 컨텍스트**: Step 3에서 수집한 기존 카테고리 목록과 태그 집계

출력:
```json
{
  "category": "폴더명",
  "tags": ["tag1", "tag2"],
  "related_notes": ["2026-03-20-xxx", ...],
  "title": "한국어 요약 제목",
  "slug": "kebab-case-slug"
}
```

**원칙**:
- 기존 카테고리와 의미가 겹치면 **기존 카테고리를 우선 재사용**
- 완전히 새로운 주제일 때만 신규 카테고리 생성 (생성 시 사용자에게 알림)
- 태그는 2~5개, 소문자-kebab-case

### Step 6: Dry-run 확인

실제 쓰기 전에 사용자에게 요약을 보여주고 확인:

```
📊 처리 요약 (profile_owner: {owner})
─────────────────────────────
수집 총:   N 건
신규:      N 건
중복 skip: N 건
merge:     N 건

🆕 신규 카테고리: [list]

📝 생성될 파일:
  - {category}/YYYY-MM-DD-{slug}.md
  ...

🔧 수정될 파일 (merge):
  - {path} (+ 추가 맥락 섹션)
  ...

진행하시겠습니까? (y/N)
```

사용자가 승인하면 Step 7로 진행.

### Step 7: Obsidian 노트 생성/갱신

**7-1. NEW 노트 생성:**

- 파일 경로: `{vault_path}/{subfolder}/{profile_owner}/{category}/YYYY-MM-DD-{slug}.md`
- 템플릿:
  ```markdown
  ---
  source: linkedin
  linkedin_urn: "urn:li:activity:..."
  activity_type: original | repost | like | comment
  profile_owner: "me 또는 username"
  profile_url: "https://www.linkedin.com/in/{username}/"
  post_author: "원글 작성자"
  source_urls:
    - https://example.com/article
  posted_at: 2026-04-05
  collected_at: 2026-04-05
  category: {category}
  tags: [tag1, tag2]
  ---

  # {title}

  ## 원글 요약
  {한국어로 2~4문장 요약}

  ## 본문
  {원글 본문 인용}

  ## 내 코멘트
  {my_comment, 없으면 섹션 생략}

  ## 포함 링크
  - [{link.title}]({link.url}) — {link.summary}

  ## 관련 노트
  - [[YYYY-MM-DD-xxx]]
  - [[YYYY-MM-DD-yyy]]

  ## 출처
  - LinkedIn: {permalink}
  ```

**7-2. MERGE 업데이트:**

기존 노트 말미에 섹션 추가 (기존 내용은 절대 수정·삭제하지 않음):

```markdown

## 추가 맥락 ({collected_at})
- 재등장 활동: {activity_type} by {profile_owner}
- permalink: {permalink}
- 새 코멘트: {my_comment}
```

frontmatter의 `source_urls[]`에 새 URL이 있으면 병합.

**7-3. MOC 갱신:**

- 카테고리 MOC: `{vault_path}/{subfolder}/{profile_owner}/{category}/_MOC.md`
- 존재하지 않으면 생성:
  ```markdown
  ---
  type: moc
  category: {category}
  profile_owner: {owner}
  ---

  # {category} MOC

  ## Notes
  ```
- 생성된 NEW 노트의 `[[파일명]]` 링크를 `## Notes` 섹션에 append (이미 있으면 skip)

### Step 8: 리포트

최종 요약을 사용자에게 출력:

```
✅ 완료

수집:  N 건
신규:  N 건 (생성됨)
merge: N 건 (갱신됨)
skip:  N 건 (중복)

📁 저장 위치: {vault_path}/{subfolder}/{profile_owner}/

📝 생성된 파일:
  - ...

🔧 수정된 파일:
  - ...

🆕 신규 카테고리:
  - ...
```

## 주의사항

- **타인 모드는 공개 프로필 정보만** 수집합니다. 로그인 계정이 볼 수 있는 범위 내에서만 동작하며, 비공개/DM은 절대 수집하지 않습니다.
- 타인 프로필 수집 시 대상 프로필을 명시하고 사용자의 **명시적 확인**을 받은 뒤 진행합니다.
- `vault_path` 외부로는 어떤 파일도 쓰지 않습니다.
- `linkedin_urn` → URL → 콘텐츠 순으로 엄격 dedup하며, merge 시 **기존 노트 본문은 수정·삭제하지 않고 섹션만 append**합니다.
- 카테고리는 매 실행마다 LLM이 결정하지만 vault의 기존 카테고리를 컨텍스트로 제공하여 일관성을 확보합니다. 신규 카테고리가 생성되면 리포트에 별도 표기합니다.
- WebFetch 실패는 치명적이지 않으며 URL만 유지한 채 진행합니다 (graceful degradation).
- LinkedIn DOM 구조 변경 시 수집 실패 가능성이 있으며, 이 경우 폴백(수동 입력) 모드로 안내합니다.
- Chrome MCP를 통한 수집이므로 토큰 사용량이 많아질 수 있습니다. 한 번에 너무 많은 activity를 수집하지 않도록 스크롤 범위를 조정합니다.

## 예시

```bash
# 본인 프로필 (기본)
/gnoyes:linkedin-knowledge-organizer

# 타인 프로필 지정
/gnoyes:linkedin-knowledge-organizer https://www.linkedin.com/in/someone/
```
