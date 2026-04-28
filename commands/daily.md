---
description: 오늘 데일리 노트를 보장한다 (없으면 생성, 어제 carry-over 적용)
---

옵시디언 볼트의 오늘 데일리 노트를 처리한다. `obsidian` 스킬의 "Workflow: 오늘 데일리 노트 보장" 절차를 그대로 따른다.

1. **오늘 / 어제 / 내일 날짜 계산**
   - `date '+%Y-%m-%d'` (오늘)
   - `date -d 'yesterday' '+%Y-%m-%d'`
   - `date -d 'tomorrow' '+%Y-%m-%d'`
   - 오늘 요일: `date '+%w'` 후 한글 매핑 (0=일, 1=월, 2=화, 3=수, 4=목, 5=금, 6=토)

2. **존재 확인**: `/mnt/c/Users/rst010/Documents/Obsidian Vault/daily/<TODAY>.md` Read 시도.

3. **이미 존재**: 그대로 두고 사용자에게 "오늘 데일리 이미 있음" 알림 + 현재 상태 (로그/Tasks 개수) 요약.

4. **존재하지 않음**:
   - 어제 데일리 (`daily/<YESTERDAY>.md`) Read 시도. 있으면 다음을 추출:
     - `## Tomorrow` 섹션의 `- [ ]` 또는 `- [x]` 항목들 (carry-over용)
     - `## Tasks` 섹션의 `- [ ]` 항목 (미완료만)
   - 새 파일 작성 (Templater 변수 모두 평가 완료 상태):
     ```markdown
     ---
     created: <TODAY>
     tags:
       - daily
     ---
     # <TODAY> (<요일>)

     ## 로그


     ## Tasks
     <carry-over 항목들 — Tomorrow 항목은 [ ] 로, 어제 미완료는 [ ] 로 그대로>

     ## Notes


     ## Tomorrow
     - [ ]

     ---
     어제: [[<YESTERDAY>]] | 내일: [[<TOMORROW>]]
     ```
   - carry-over가 없으면 Tasks는 `- [ ]` 빈 항목 1개.

5. **결과 보고**: 생성 여부, carry-over된 항목 수, 파일 경로를 짧게 (3줄 이내) 알린다.
