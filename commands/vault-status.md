---
description: 옵시디언 볼트 현황을 한눈에 본다
---

볼트 상태 요약을 표시한다.

볼트: `/mnt/c/Users/rst010/Documents/Obsidian Vault`

다음 4가지를 병렬로 수집해서 한 화면에 정리:

1. **오늘 데일리**
   - `daily/<TODAY>.md` 존재 여부
   - 있으면: `## Tasks` 미완료 개수, `## 로그` 항목 개수
   - 없으면: "오늘 데일리 미생성"

2. **진행 중 프로젝트**
   - `projects/*.md` 중 frontmatter `status: active` 인 것 목록 (파일명만, 최대 10개)

3. **최근 노트** (5개)
   - `notes/*.md` 를 mtime 역순으로 5개 (파일명 + 날짜)

4. **Git 상태**
   - `git status --short` 결과 (변경 파일 수)
   - `git log -1 --oneline` (최근 커밋)

출력 형식 (간결):
```
## 오늘 (<DATE>)
- 데일리: <상태>
- Tasks 미완료: N
- 로그: N건

## 활성 프로젝트
- ...

## 최근 노트
- ...

## Git
- 변경: N개
- 최근 커밋: <hash> <msg>
```

사용자가 "어떻게 할까?" 묻기 전에 출력만 하고 끝낸다 (액션 없음).
