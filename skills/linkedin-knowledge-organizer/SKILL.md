---
name: linkedin-knowledge-organizer
description: LinkedIn 활동(퍼온글/좋아요/코멘트/원글)을 Obsidian 지식 데이터베이스에 인사이트 중심으로 분석·저장합니다. 본인 프로필은 기본값이며 인자로 타인 프로필 URL을 넘기면 해당 사용자 활동도 정리 가능. 링크드인 지식정리, 퍼온글 정리, Obsidian knowledge base, LinkedIn organize
---

# LinkedIn Knowledge Organizer

LinkedIn activity(본인 또는 지정한 타인 프로필)의 원글·공유(Repost)·좋아요·코멘트를 수집하여, 본문과 포함 링크의 내용을 **깊이 분석**한 뒤 Obsidian 지식 데이터베이스에 **인사이트 중심의 atomic note**로 저장합니다.

**핵심 원칙**: 단순 요약이 아니라, 나중에 글을 쓰거나 의사결정할 때 꺼내볼 수 있는 **살아있는 지식**을 만든다.

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
        "subfolder": "03_Resources/LinkedIn"
      }
    }
  }
  ```
- WebFetch 도구 사용 가능

## Vault 구조

```
{vault_path}/{subfolder}/
├── _Index.md                          ← 전체 인덱스 (카테고리별 + 최근순)
├── concepts/                          ← 개념 노트 (atomic concept)
│   ├── AI-코딩-에이전트.md
│   ├── RAG-정확도-트레이드오프.md
│   └── 1인-풀스택-운영.md
├── {category}/                        ← 카테고리별 폴더
│   ├── _MOC.md                        ← 카테고리 Map of Content
│   ├── YYYY-MM-DD-{slug}.md           ← 개별 지식 노트
│   └── ...
└── ...
```

## 실행 워크플로우

### Step 1: 환경 확인 및 모드 결정

1. `mcp__claude-in-chrome__tabs_context_mcp`로 Chrome MCP 연결 확인
   - 실패 시 → **Step 1-B 폴백 모드**
2. `~/.claude/settings.json`을 Read로 로드
   - `gnoyes.linkedin_organizer.vault_path` 미설정 시: 설정 방법 안내 후 중단
   - `subfolder` 기본값: `03_Resources/LinkedIn`
3. `vault_path` 디렉토리 존재 여부 및 쓰기 가능 여부 확인
4. **모드 결정**:
   - `$ARGUMENTS` 없음 → **본인 모드**
     - 대상 URL: `https://www.linkedin.com/in/me`
     - `profile_owner = "me"`
   - `$ARGUMENTS`가 `https://www.linkedin.com/in/{username}/?` 패턴에 매칭 → **타인 모드**
     - username 추출, `profile_owner = username`
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
  "post_author_title": "작성자의 직함/소개",
  "text": "원글 본문 (전문)",
  "my_comment": "본인이 남긴 코멘트 (해당 시)",
  "urls": ["https://..."],
  "posted_at": "YYYY-MM-DD",
  "permalink": "https://www.linkedin.com/feed/update/..."
}
```

### Step 3: Vault 스캔 (중복 인덱스 + 컨텍스트)

1. `{vault_path}/{subfolder}/**/*.md` 순회하여 frontmatter 파싱 (Glob + Read)
2. **중복 인덱스 구축**:
   - `linkedin_urn` → 파일 경로 맵
   - 정규화된 URL(utm_*, fbclid, gclid 제거, trailing slash 정규화) → 파일 경로 맵
3. **기존 지식 컨텍스트** 수집:
   - 기존 카테고리 폴더명 목록
   - 기존 노트들의 `tags`, `category`, `concepts` frontmatter 값 집계
   - 기존 개념 노트(`concepts/`) 목록 — 새 노트가 기존 개념과 연결될 수 있도록
   - `_MOC.md` 파일 목록

### Step 4: 링크 보강 (WebFetch)

각 수집 항목의 `urls[]`에 대해:

1. WebFetch로 페이지 가져오기 (실패·타임아웃은 graceful하게 스킵)
2. 추출: `{ url, title, summary(1-2문단), domain, key_points[] }`
3. 결과를 항목의 `link_context[]`에 저장. 분석 단계 입력으로 사용

### Step 5: 중복 판정 및 깊이 분석

**5-1. 중복 판정 (항목별):**

1. `linkedin_urn`이 인덱스에 존재 → **SKIP** (이미 저장됨)
2. 정규화 URL 중 하나라도 인덱스에 존재 → **MERGE**
   - 기존 노트에 `## 추가 맥락 ({collected_at})` 섹션을 append
3. 그 외 → **NEW**

**5-2. 깊이 분석 (NEW 항목에 대해 — 이것이 핵심):**

각 NEW 항목에 대해 아래를 도출한다. **단순 요약이 아니라 지식으로서의 가치를 추출**하는 것이 목적이다.

입력:
- 원글 본문 (전문)
- 본인 코멘트 (있을 경우)
- `link_context[]` (제목 + 요약 + 핵심 포인트)
- 작성자의 직함/배경 (권위·맥락 판단용)
- **Vault 컨텍스트**: 기존 카테고리, 태그, 개념 노트 목록

분석하여 도출할 것:

```json
{
  "category": "폴더명",
  "title": "인사이트 중심 한국어 제목 (게시글 제목이 아닌, 핵심 인사이트를 담은 제목)",
  "slug": "kebab-case-slug",
  "tags": ["tag1", "tag2", "tag3"],
  "concepts": ["개념1", "개념2"],
  "core_insight": "이 글의 핵심 인사이트 (1-2문장, 나중에 이것만 봐도 가치를 알 수 있도록)",
  "key_takeaways": ["구체적 교훈/원칙 1", "구체적 교훈/원칙 2", ...],
  "so_what": "내 상황(개발자/창업/커리어)에 어떻게 적용할 수 있는가",
  "thinking_questions": ["이 인사이트에서 파생되는 생각거리 질문"],
  "related_existing_notes": ["기존 노트 파일명", ...],
  "evidence_quality": "경험담 | 데이터기반 | 의견 | 사례연구 | 프레임워크"
}
```

**카테고리 원칙**:
- 기존 카테고리와 의미가 겹치면 **기존 카테고리를 우선 재사용**
- 완전히 새로운 주제일 때만 신규 카테고리 생성
- 태그는 3~7개, 소문자-kebab-case. 기존 태그 재사용 우선
- `concepts[]`: 이 글에서 다루는 핵심 개념 2~3개. 기존 개념 노트가 있으면 정확히 같은 이름 사용

**제목 작성 원칙**:
- ❌ "Claude Code Skills로 1인 풀스택 운영" (게시글 제목 복사)
- ✅ "소수 정예 팀이 AI 도구로 풀스택 운영하는 전략" (인사이트 중심)
- 나중에 이 제목만 보고도 "아, 이 지식이 필요해"라고 판단할 수 있어야 한다

### Step 6: Dry-run 확인

실제 쓰기 전에 사용자에게 요약을 보여주고 확인:

```
📊 처리 요약
─────────────────────────────
수집 총:   N 건
신규:      N 건
중복 skip: N 건
merge:     N 건

📝 생성될 지식 노트:
  1. {category}/{slug} — "{core_insight 요약}"
  2. ...

💡 생성/갱신될 개념 노트:
  - [[개념1]] (신규)
  - [[개념2]] (기존 — 연결 추가)

🆕 신규 카테고리: [list, 있을 경우만]

진행하시겠습니까? (y/N)
```

사용자가 승인하면 Step 7로 진행.

### Step 7: Obsidian 노트 생성/갱신

**7-1. 지식 노트 생성 (NEW):**

- 파일 경로: `{vault_path}/{subfolder}/{category}/YYYY-MM-DD-{slug}.md`
- **반드시 게시글 1건당 1개의 개별 파일**로 생성한다. 절대로 여러 게시글을 하나의 파일에 모으지 않는다.

```markdown
---
source: linkedin
linkedin_urn: "urn:li:activity:..."
activity_type: original | repost | like | comment
post_author: "원글 작성자"
post_author_title: "작성자 직함"
source_urls:
  - https://example.com/article
posted_at: 2026-04-05
collected_at: 2026-04-05
category: "{category}"
tags: [tag1, tag2, tag3]
concepts: [개념1, 개념2]
evidence_type: "경험담 | 데이터기반 | 의견 | 사례연구 | 프레임워크"
---

# {인사이트 중심 제목}

## 핵심 인사이트

{이 글에서 가져갈 가장 중요한 깨달음. 2-3문장으로 명확하게.
 나중에 이 섹션만 훑어봐도 이 노트의 가치를 즉시 판단할 수 있어야 한다.}

## 주요 논점

{원글의 핵심 주장과 근거를 구조화하여 정리.
 단순 요약이 아니라, 주장-근거-사례의 논리 흐름을 보존한다.}

- **주장 1**: ...
  - 근거/사례: ...
- **주장 2**: ...
  - 근거/사례: ...

## 실천 포인트

{내가 구체적으로 적용할 수 있는 액션 아이템이나 원칙.
 "그래서 나는 뭘 하면 되는가?"에 답한다.}

- [ ] 액션/원칙 1
- [ ] 액션/원칙 2

## 생각거리

{이 글을 읽고 떠오르는 질문이나 추가 탐구할 주제.
 나중에 글을 쓰거나 의사결정할 때 이 질문들이 사고를 확장해준다.}

- ...

## 연결

- **관련 개념**: [[개념1]] · [[개념2]]
- **관련 노트**: [[YYYY-MM-DD-xxx]] · [[YYYY-MM-DD-yyy]]
{기존 vault의 다른 노트와 연결점이 있으면 추가}

## 원문

> {원글 본문 전문을 인용 블록으로}

**작성자**: {post_author} ({post_author_title})
**출처**: [LinkedIn 원문]({permalink})

{my_comment이 있을 경우:}
### 내 코멘트
> {my_comment}

{link_context가 있을 경우:}
### 참고 링크
- [{link.title}]({link.url}) — {link.summary}
```

**7-2. 개념 노트 생성/갱신 (`concepts/`):**

개념 노트는 여러 지식 노트를 관통하는 **하나의 핵심 개념**을 추적하는 허브 노트다.

- 파일 경로: `{vault_path}/{subfolder}/concepts/{개념명}.md`
- 해당 개념이 이미 존재하면 → `## 관련 지식 노트` 섹션에 새 링크 append
- 존재하지 않으면 → 새로 생성:

```markdown
---
type: concept
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [concept, {관련태그}]
---

# {개념명}

{이 개념에 대한 1-2문장 정의. 수집된 글들에서 추출한 핵심 의미.}

## 관련 지식 노트

- [[YYYY-MM-DD-{slug}]] — {해당 노트의 core_insight 한 줄}
```

**개념 노트 작성 원칙**:
- 너무 넓은 개념은 피한다 (❌ "AI", "개발" → ✅ "AI-코딩-에이전트", "RAG-정확도-트레이드오프")
- 여러 글에서 반복 등장하는 주제를 개념으로 승격한다
- 개념 정의는 수집된 글들의 관점을 종합하여 작성한다

**7-3. MERGE 업데이트:**

기존 노트 말미, `## 원문` 섹션 직전에 추가 (기존 내용은 절대 수정·삭제하지 않음):

```markdown

## 추가 맥락 ({collected_at})

- 재등장 활동: {activity_type}
- 새 코멘트: {my_comment}
- [LinkedIn 원문]({permalink})
```

frontmatter의 `source_urls[]`에 새 URL이 있으면 병합.

**7-4. MOC 갱신:**

- 카테고리 MOC: `{vault_path}/{subfolder}/{category}/_MOC.md`
- 존재하지 않으면 생성:
  ```markdown
  ---
  type: moc
  category: "{category}"
  updated: YYYY-MM-DD
  ---

  # {category}

  {이 카테고리에 속하는 지식 노트들의 Map of Content.}

  ## 노트 목록
  ```
- 생성된 NEW 노트의 `[[파일명]]` 링크를 `## 노트 목록` 섹션에 append (이미 있으면 skip)
- 형식: `- [[YYYY-MM-DD-{slug}]] — {core_insight 한 줄 요약}`

**7-5. 전체 인덱스 갱신:**

- 파일 경로: `{vault_path}/{subfolder}/_Index.md`
- 존재하지 않으면 생성, 있으면 갱신:

```markdown
---
type: index
updated: YYYY-MM-DD
---

# LinkedIn 지식 데이터베이스

LinkedIn 활동에서 추출한 인사이트를 모은 지식 저장소입니다.

## 카테고리

{각 카테고리별 MOC 링크와 노트 수}
- [[{category}/_MOC|{category}]] ({N}개 노트)

## 최근 추가

{최근 10건의 노트를 날짜 역순으로}
- {posted_at} [[YYYY-MM-DD-{slug}|{title}]]

## 개념 맵

{개념 노트들을 알파벳/가나다 순으로}
- [[concepts/{개념명}|{개념명}]]
```

### Step 8: 리포트

최종 요약을 사용자에게 출력:

```
✅ 완료

수집:  N 건
신규:  N 건 (지식 노트 생성)
merge: N 건 (기존 노트 보강)
skip:  N 건 (중복)

💡 개념 노트: N 개 생성, N 개 갱신

📁 저장 위치: {vault_path}/{subfolder}/

📝 생성된 지식 노트:
  - {category}/YYYY-MM-DD-{slug} — "{core_insight}"
  ...

🔗 개념 노트:
  - concepts/{개념명} (신규/갱신)
  ...
```

## 주의사항

- **게시글 1건 = 개별 파일 1개**: 절대로 여러 게시글을 하나의 파일에 모아쓰지 않는다. 이것이 지식 데이터베이스의 핵심이다.
- **인사이트 우선**: "무슨 글이었는가"보다 "이 글에서 무엇을 배울 수 있는가"에 집중한다.
- **타인 모드는 공개 프로필 정보만** 수집합니다. 비공개/DM은 절대 수집하지 않습니다.
- 타인 프로필 수집 시 대상 프로필을 명시하고 사용자의 **명시적 확인**을 받은 뒤 진행합니다.
- `vault_path` 외부로는 어떤 파일도 쓰지 않습니다.
- `linkedin_urn` → URL → 콘텐츠 순으로 엄격 dedup하며, merge 시 **기존 노트 본문은 수정·삭제하지 않고 섹션만 append**합니다.
- 카테고리와 개념은 vault의 기존 구조를 컨텍스트로 제공하여 일관성을 확보합니다.
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
