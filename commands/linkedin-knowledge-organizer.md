---
description: LinkedIn 활동을 Obsidian 지식 저장소에 카테고리화하여 정리합니다
---

# LinkedIn Knowledge Organizer

LinkedIn 활동(원글/공유/좋아요/코멘트)을 수집하여 Obsidian vault에 중복 없이 카테고리화하여 저장합니다.

## 사용법

기본은 본인 프로필입니다. 타인 프로필 URL을 인자로 전달하면 해당 사용자의 공개 활동을 정리합니다.

## 워크플로우

1. Chrome MCP로 LinkedIn 활동 페이지 데이터 수집
2. 포함 링크의 내용을 WebFetch로 보강
3. Vault 기존 노트 스캔으로 중복 판정 (URN/URL 기반)
4. 신규 항목 카테고리화 및 Obsidian 노트 생성
5. MOC(Map of Content) 갱신 및 위키링크 연결
6. Dry-run 확인 후 저장
