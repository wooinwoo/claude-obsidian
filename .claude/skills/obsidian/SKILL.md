---
name: obsidian
description: Manage the user's Obsidian vault at /mnt/c/users/rst010/documents/obsidian vault. Use ONLY when the user has clear intent to write to / edit / read from / organize Obsidian notes — e.g. "오늘 데일리에 ~", "옵시디언/볼트에 ~ 메모/적어/기록", "~ 끝냈어 (체크)", "어제/이번주 뭐 했지", explicit /daily / /note / /vault-* commands, or unmistakable vault-targeted note-taking context. Do NOT trigger for casual mentions ("옵시디언 좋다"), generic note-taking not aimed at the vault, or other files in other directories. The user never edits Obsidian directly; every change goes through this skill.
---

# Obsidian Vault Skill

## Vault location

```
/mnt/c/users/rst010/documents/obsidian vault
```

이 경로 외에는 옵시디언 작업 아님. 항상 이 경로 기준.

## Folder map

```
daily/        YYYY-MM-DD.md     날짜별 로그
projects/     <프로젝트명>.md    끝 시점 있는 큰 작업
notes/        <주제>.md          그 외 다 (메모 / 지식 / 참고)
archive/      안 보는 것
_templates/   Templater 템플릿
```

위치 결정 (모호하면 `notes/`):
- 시간 기반 로그 ("오늘 ~", "지금 ~", "방금 ~", "X시에 ~") → `daily/<TODAY>.md`
- 끝 시점 있는 큰 작업 → `projects/`
- 짧은 메모, 지식, 참고자료 → `notes/`
- 끝난 프로젝트 / 안 보는 것 → `archive/` (사용자 동의 후에만 이동)

## 핵심 원칙

1. **사용자는 옵시디언 직접 편집 안 함** — 모든 변경은 이 스킬 경유.
2. **Templater 변수 직접 평가** — 파일 생성 시 `<% tp.date.now(...) %>` 같은 코드를 그대로 쓰지 말고 실제 값으로 치환해서 작성. Claude가 직접 파일 쓰기라 Templater 플러그인이 평가할 기회가 없음.
3. **현재 시간이 필요할 때**: `date '+%H:%M'` (bash) 실행해서 얻음. Claude는 시계 없음.
4. **현재 날짜**: 시스템 컨텍스트의 `currentDate` 사용. 의심되면 `date '+%Y-%m-%d'`.
5. **시간대 확인**: 첫 시간 기록 전에 `date '+%Z %z'` 실행. KST(`+0900`)가 아니면 사용자에게 알리고 멈춤. WSL이 UTC 기본이라 9시간 어긋날 수 있음. 사용자가 KST가 아니라고 확인해주면 진행.

## 파일명 / 형식 규칙

- 한글 / 공백 OK. 케밥/스네이크 안 씀.
- 데일리: `YYYY-MM-DD.md` (예: `2026-04-28.md`)
- 프로젝트 / 노트: 자연 제목 (예: `Q2 발표자료.md`, `Docker 메모.md`)
- Frontmatter 최소: `created: YYYY-MM-DD`, `tags: [...]`
- 이모지 헤더 금지. H1은 파일당 1개.
- 시간: 24시간 (`14:30`, `09:05`)
- 내부 링크: `[[wikilink]]` 우선
- 태그: 소문자 + 하이픈, 계층형 가능 (`project/foo`)

## Workflow: 오늘 데일리 보장 (idempotent)

사용자가 "오늘 ~", "지금 ~", "방금 ~" 류로 적으라고 하면 먼저 오늘 데일리가 있는지 확인.

1. 경로: `daily/<TODAY>.md` (TODAY = `currentDate`)
2. **존재 확인**: Read 시도.
3. **존재 → carry-over 절대 다시 안 함**. 그대로 사용. (idempotency 가드)
4. **존재 안 함 → 신규 생성**:
   - 어제 데일리 (`daily/<YESTERDAY>.md`) 있으면 추출:
     - `## Tomorrow` 섹션의 모든 항목 → 오늘 `## Tasks` 로 (체크박스 상태 유지)
     - `## Tasks` 의 미완료 (`- [ ]`) → 오늘 `## Tasks` 로 추가
   - 어제 데일리 없거나 carry-over 항목 0개 → Tasks는 빈 `- [ ]` 1개
   - 새 파일 작성 (Templater 변수 모두 치환):

```markdown
---
created: <TODAY>
tags:
  - daily
---
# <TODAY> (<요일>)

## 로그


## Tasks
<carry-over 항목들 또는 - [ ]>

## Notes


## Tomorrow
- [ ]

---
어제: [[<YESTERDAY>]] | 내일: [[<TOMORROW>]]
```

요일 한글: `date '+%w'` (0=일, 1=월, 2=화, 3=수, 4=목, 5=금, 6=토).

## Workflow: 시간 로그 추가

"지금 X 했어 / X 잡혔어 / 방금 X / X시에 ~" 류:

1. 오늘 데일리 보장.
2. `date '+%H:%M'` 으로 현재 시각.
3. `## 로그` 섹션 끝에 `- HH:MM <내용>` Edit으로 추가.
4. 결과 한 줄 보고: "로그 추가됨: HH:MM <내용>".

여러 시각의 일이 한 번에 들어오면 (예: "9시에 회의, 10시에 코드리뷰") 각각 별도 줄로.

## Workflow: 태스크 추가

"X 해야 해 / 할 일 X / X 잊지 마":
- 데일리의 `## Tasks` 끝에 `- [ ] <내용>` 추가.
- 명시적으로 "프로젝트 Y의" 라고 하면 `projects/Y.md` 의 `## Tasks > In Progress` 또는 `Backlog`.

## Workflow: 태스크 완료 토글

"X 끝냈어 / X 완료 / X 했음":

1. 검색 순서:
   - 오늘 데일리 `## Tasks` 의 `- [ ] *X*`
   - 활성 프로젝트들 (`projects/*.md` frontmatter `status: active`) 의 미완료 태스크
2. 매칭 후보 정리:
   - 1개 → 바로 토글
   - 2개 이상 → 사용자에게 후보 제시하고 선택받기
   - 0개 → 사용자에게 알리고 새 항목으로 추가할지 묻기 (오늘 데일리에 `- [x] X ✅ HH:MM` 으로 후행 기록).
3. 토글 형식: `- [x] <원래 내용> ✅ HH:MM`
4. (선택) `## 로그`에도 한 줄 완료 기록 — 사용자 톤이 짧고 사무적이면 생략, 회고 톤이면 추가.

## Workflow: 노트 작성

"X 메모해 / X 정리해 / X 적어둬" (시간 로그 아님):

1. **위치 결정**: Folder map 기준. 모호하면 `notes/`.
2. **파일명 결정**: 자연 제목. 기존 파일과 동일 제목 검사 (Glob 또는 Read 시도).
   - 동일 파일 있음 → 사용자에게 물음: "기존 파일에 추가" / "새 파일 (이름 다르게)" / "덮어쓰기".
3. **템플릿 베이스**:
   - `projects/` → `_templates/project.md`
   - `notes/` → `_templates/note.md`
   - Templater 변수 모두 치환.
4. **Frontmatter**:
   - `created`: 오늘 날짜
   - `tags`: 인자에서 추론 1~3개 (예: "Docker 메모" → `[note, docker]`). 추론 안 되면 `[note]` 또는 `[project]`.
5. **본문**: 짧은 주제어면 빈 본문, 긴 내용이면 정리해서 작성. 관련 노트는 `[[wikilink]]`.
6. 결과: 파일 경로 1줄 + 주요 frontmatter 1줄.

## Workflow: 회고 / 검색

- "어제 뭐 했지?" → `daily/<YESTERDAY>.md` 읽고 `## 로그` 위주로 요약.
- "이번 주 한 일" → `daily/` 에서 최근 7일 파일 mtime 정렬, 합쳐서 핵심만 요약.
- "이번 달 X 관련" → `notes/`, `projects/` 에서 mtime 30일 이내 + 본문에 X 포함 파일 검색.

## Examples (입력 → 행동)

**입력**: "지금 김부장이 회의 잡았어 — Q2 발표자료"
**행동**:
1. `date '+%H:%M'` → 예: `14:30`
2. 오늘 데일리 보장 (없으면 carry-over로 생성, 있으면 그대로)
3. `## 로그` 끝에 `- 14:30 김부장 회의 — Q2 발표자료` 추가
4. 보고: "로그 추가됨: 14:30 김부장 회의 — Q2 발표자료"

**입력**: "발표자료 끝냈어"
**행동**:
1. 오늘 데일리에서 `- [ ]` 중 "발표자료" 매칭 검색
2. 1개 발견 → `- [x] 발표자료 1차 완성 ✅ 16:00` 으로 토글 (현재 시각)
3. 0개면 활성 프로젝트들에서 검색
4. 보고: "✓ 발표자료 1차 완성 (오늘 데일리)"

**입력**: "Docker 컨테이너 자주 까먹는 거 메모해둘래"
**행동**:
1. 위치: `notes/` (지식)
2. 파일명: `Docker 컨테이너.md` 또는 `Docker 메모.md` — 사용자 표현에 맞춤
3. 기존 동명 파일 검사 → 없음
4. `_templates/note.md` 베이스로 frontmatter (`tags: [note, docker]`) + H1만 작성
5. 보고: "생성됨: notes/Docker 컨테이너.md"

**입력**: "어제 뭐 했지?"
**행동**:
1. `daily/<YESTERDAY>.md` Read
2. `## 로그`, `## Tasks` (완료된 것 위주), `## Notes` 추출
3. 5~10줄로 요약해서 답변. 파일 수정 없음.

**입력**: "내일 오후 2시에 김부장 1on1"
**행동**:
1. `daily/<TOMORROW>.md` 존재 확인.
2. **있음** → `## 로그` 또는 `## Tasks` 에 `- 14:00 김부장 1on1` 추가 (시각 명시이고 일정이라 로그가 자연스러움).
3. **없음** → 오늘 데일리의 `## Tomorrow` 에 `- 14:00 김부장 1on1` 추가. (내일 데일리 미리 만들지 않음 — 내일 첫 액션 때 carry-over로 자동 들어감.)

**입력**: "옵시디언 좋네" 또는 "이거 메모해두자" (코드 리뷰 중, 옵시디언 언급 없음)
**행동**: 이 스킬 트리거 안 함. 옵시디언 의도가 명확하지 않음.

## Failure modes

- **볼트 경로 없음** → 멈춤. 사용자에게 vault location 확인 요청.
- **템플릿 파일 없음** (`_templates/daily.md` 등) → 멈춤. 알림.
- **시간대 의심** (KST 아님) → 멈춤. 사용자 확인 후 진행.
- **어제 데일리 형식 깨짐** (`## Tomorrow` / `## Tasks` 섹션 없음) → carry-over 스킵, 사용자에게 1줄 알림 후 빈 데일리로 생성.
- **태스크 매칭 모호** → 후보 제시, 사용자 선택 대기.
- **사용자가 옵시디언 앱에서 동시 편집 중** (충돌 의심) → 의심되면 사용자에게 알림. 보통 사용자는 직접 편집 안 한다는 전제.

## 절대 안 함

- `<% tp.* %>` Templater 코드 그대로 파일에 쓰기 (반드시 평가)
- 사용자 동의 없이 `archive/` 로 이동하거나 파일 삭제
- 시간 추측 (반드시 `date` 호출)
- 옵시디언 외 경로에 옵시디언 노트 만들기
- 한 번 쓴 데일리에 carry-over 다시 적용
- `git push --force` 또는 `git reset --hard`

## 시작 체크 (해당 세션의 첫 옵시디언 액션 직전 1회)

1. 볼트 경로 존재? (Read 또는 Glob 시도) — 없으면 멈춤.
2. 4개 폴더 + `_templates/` 존재? — 없으면 알림.
3. `date '+%Z %z'` — KST(`+0900`)가 아니면 사용자에게 알리고 진행 여부 확인.
