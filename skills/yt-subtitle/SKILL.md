---
name: yt-subtitle
description: YouTube 영상의 자막을 추출하여 마크다운 파일로 저장합니다. 유튜브 자막, YouTube 자막 추출, subtitle extract
---

# YouTube Subtitle Extractor

YouTube 영상 링크를 받아 자막을 추출하고 YAML frontmatter가 포함된 마크다운 파일로 저장합니다.

## 인자

- `$ARGUMENTS`: YouTube URL (필수). 언어 코드를 추가할 수 있음 (예: `URL ko`, `URL en`)

## 사전 요구사항

- `yt-dlp` 설치 필요 (`brew install yt-dlp`)
- `ffmpeg` 설치 권장 (`brew install ffmpeg`) - 없어도 VTT 직접 파싱으로 동작

## 실행 워크플로우

### Step 1: yt-dlp 설치 확인

```bash
which yt-dlp
```

미설치 시 사용자에게 `brew install yt-dlp` 실행 여부를 확인한 후 설치합니다.

### Step 2: 인자 파싱

`$ARGUMENTS`에서 YouTube URL과 언어 코드를 분리합니다.

- 첫 번째 인자: YouTube URL
- 두 번째 인자 (선택): 언어 코드 (ko, en, ja 등)

언어 코드가 없으면 사용 가능한 자막 목록을 조회하여 사용자에게 선택하게 합니다:

```bash
yt-dlp --list-subs --skip-download "URL"
```

### Step 3: 메타데이터 추출

```bash
yt-dlp --skip-download --print-json "URL" 2>/dev/null | python3 -c "
import json, sys
d = json.load(sys.stdin)
print(json.dumps({'title': d.get('title',''), 'channel': d.get('channel',''), 'upload_date': d.get('upload_date',''), 'duration_string': d.get('duration_string','')}, ensure_ascii=False))
"
```

### Step 4: 자막 다운로드

임시 디렉토리에 자막 파일을 다운로드합니다:

```bash
yt-dlp --skip-download --write-sub --write-auto-sub --sub-lang "LANG" --sub-format vtt -o "/tmp/yt-sub-%(id)s" "URL"
```

- `--write-sub`으로 수동 자막 우선
- 없으면 `--write-auto-sub`으로 자동생성 자막 사용
- ffmpeg가 있으면 `--convert-subs srt` 추가

### Step 5: 자막 파싱 및 정리

Python3으로 VTT/SRT 파일을 파싱합니다:

1. VTT 헤더 제거
2. 타임스탬프 라인 제거
3. 시퀀스 번호 제거
4. HTML 태그 제거 (`<c>`, `</c>` 등)
5. VTT 포지셔닝 지시자 제거
6. 중복 라인 제거 (자동생성 자막에서 흔함)
7. 줄바꿈된 문장을 하나로 합침
8. 문장 부호 기준으로 문단 구분 (약 5문장씩)

### Step 6: 마크다운 파일 생성

현재 작업 디렉토리에 저장합니다.

**파일명**: 영상 제목에서 특수문자(`/\:*?"<>|`)를 제거하여 `영상제목.md`

**형식**:

```markdown
---
title: "영상 제목"
channel: "채널명"
url: "https://youtube.com/watch?v=xxx"
language: "ko"
duration: "12:34"
date: YYYY-MM-DD
---

# 영상 제목

자막 본문 텍스트 (문단 단위로 정리)
```

### Step 7: 정리

- `/tmp`의 임시 자막 파일 삭제
- 생성된 파일 경로와 요약 정보(파일명, 언어, 줄 수) 출력

## 주의사항

- 자막이 없는 영상은 자동생성 자막을 시도하고, 그것도 없으면 사용자에게 안내
- 이미 같은 이름의 파일이 있으면 덮어쓰기 전에 사용자 확인
- 매우 긴 영상(1시간+)은 파싱에 시간이 걸릴 수 있음

## 예시

```bash
# 영어 자막 추출
/gnoyes:yt-subtitle https://www.youtube.com/watch?v=dQw4w9WgXcQ en

# 한국어 자막 추출
/gnoyes:yt-subtitle https://youtu.be/dQw4w9WgXcQ ko

# 언어 선택 (대화형)
/gnoyes:yt-subtitle https://www.youtube.com/watch?v=dQw4w9WgXcQ
```
