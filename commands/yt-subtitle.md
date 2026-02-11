---
description: YouTube 영상의 자막을 추출하여 마크다운 파일로 저장합니다
---

# YouTube Subtitle Extractor

YouTube 영상 링크를 받아 자막을 추출하고 마크다운 파일로 저장합니다.

## 사용법

YouTube URL과 선택적으로 언어 코드를 전달합니다.

## 워크플로우

1. `yt-dlp` 설치 확인 (미설치 시 `brew install yt-dlp` 안내)
2. URL에서 영상 메타데이터(제목, 채널명) 추출
3. 지정 언어의 자막 다운로드 (수동 자막 우선, 없으면 자동생성 자막)
4. 자막 파싱: 타임스탬프 제거, 중복 제거, 문단 정리
5. YAML frontmatter + 본문 형식의 마크다운 파일 생성
6. 현재 디렉토리에 `영상제목.md`로 저장
