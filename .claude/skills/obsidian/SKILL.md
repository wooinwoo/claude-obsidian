---
name: obsidian
description: Manage user's Obsidian vault — create/edit daily notes, projects, and notes; toggle tasks; log timestamped events. Triggers when the user mentions Obsidian, vault (볼트), daily/일지, "메모해/적어줘/기록해" in note-taking context, or asks to read/write/organize anything in the vault. The user never edits Obsidian directly; all changes go through this skill.
---

# Obsidian Vault Skill

## Vault location

```
/mnt/c/Users/rst010/Documents/Obsidian Vault
```

이 경로 외에는 옵시디언 작업이 아님. 항상 이 경로를 기준으로 동작.

## Folder map

```
daily/        YYYY-MM-DD.md  날짜별 로그
projects/     <프로젝트명>.md  진행 중인 큰 작업
notes/        <주제>.md  메모 / 지식 / 참고 (그 외 다)
archive/      안 보는 것
_templates/   Templater 템플릿
```

**위치 결정 (모호하면 `notes/`)**:
- 시간 기반 로그 ("오늘 ~", "지금 ~", "방금 ~") → `daily/오늘.md`
- 끝 시점 있는 큰 작업 → `projects/`
- 짧은 메모, 지식, 참고자료 → `notes/`
- 끝난 프로젝트 / 안 보는 것 → `archive/` (이동 시점에만)

## 핵심 원칙

1. **사용자는 옵시디언 직접 편집 안 함** — 모든 변경은 이 스킬 경유.
2. **Templater 변수 직접 평가** — 파일 생성 시 `<% tp.date.now(...) %>` 같은 코드를 그대로 쓰지 말고 실제 값으로 치환해서 작성. 이유: Claude는 파일시스템 직접 쓰기이므로 Templater 플러그인이 변수를 평가할 기회가 없음.
3. **현재 시간이 필요할 때**: `date '+%H:%M'` (bash) 실행해서 얻음. Claude는 시계가 없음.
4. **현재 날짜**: 시스템 컨텍스트의 `currentDate` 사용. 의심되면 `date '+%Y-%m-%d'`.

## 파일명 규칙

- 한글 / 공백 OK. 케밥/스네이크 케이스 안 씀.
- 데일리: `YYYY-MM-DD.md` (예: `2026-04-28.md`)
- 프로젝트/노트: 자연스러운 제목 (예: `Q2 발표자료.md`, `Docker 메모.md`)

## Frontmatter 규칙

최소만 강제:
- `created: YYYY-MM-DD`
- `tags: [...]` (1개 이상)

선택: `updated`, `aliases`, `status` (프로젝트의 경우 `active`/`done`/`paused`)

이모지 헤더 금지. H1은 파일당 1개 (제목과 동일).

## 시간 / 링크 / 태그 규칙

- 시간: 24시간 (`14:30`, `09:05`)
- 내부 링크: `[[wikilink]]` 우선
- 태그: 소문자, 하이픈, 계층형 가능 (`project/foo`, `area/wsl`)

## Workflow: 오늘 데일리 노트 보장

사용자가 "오늘 ~" 류로 적으라고 하면 먼저 오늘 데일리 노트가 있는지 확인하고 없으면 생성.

1. 경로 계산: `daily/<TODAY>.md` (TODAY = `currentDate`)
2. 존재 확인 (Read 시도, 없으면 생성)
3. 신규 생성 시 템플릿 채우기 (아래 "신규 데일리 작성" 참고)
4. carry-over: 어제 데일리(`daily/<YESTERDAY>.md`)가 있으면 그 안의:
   - `## Tomorrow` 항목 → 오늘의 `## Tasks` 로 복사
   - `## Tasks` 중 미완료 (`- [ ]`) → 오늘의 `## Tasks` 로 복사
   복사 후 어제 데일리는 손대지 않음 (히스토리 보존).

### 신규 데일리 템플릿 (Templater 변수 평가 후)

```markdown
---
created: <TODAY>
tags:
  - daily
---
# <TODAY> (<요일 한글>)

## 로그


## Tasks
- [ ]

## Notes


## Tomorrow
- [ ]

---
어제: [[<YESTERDAY>]] | 내일: [[<TOMORROW>]]
```

요일 한글 매핑: 일/월/화/수/목/금/토. `date '+%w'` (0=일) 또는 `date '+%A'` 후 매핑.

## Workflow: 시간 로그 추가

"지금 X 했어 / X 잡혔어 / 방금 X" 류:

1. 오늘 데일리 보장 (위 워크플로)
2. `date '+%H:%M'` 으로 현재 시각 획득
3. `## 로그` 섹션 끝에 `- HH:MM <내용>` 추가 (Edit 사용, `## 로그\n` 다음 줄에 append)

## Workflow: 태스크 추가 / 토글

**추가** ("X 해야 해 / 할 일 X"):
- 데일리의 `## Tasks` 끝에 `- [ ] <내용>` 추가
- 프로젝트 관련이면 해당 프로젝트 노트 `## Tasks > In Progress` 또는 `Backlog` 에 추가

**완료** ("X 끝냈어 / 완료"):
1. 후보 태스크 검색: 오늘 데일리 → 진행 중 프로젝트들 순으로 `- [ ] <X와 매칭>` 찾기
2. 찾으면 토글: `- [x] <내용> ✅ HH:MM`
3. 동시에 `## 로그`에 완료 기록 1줄 추가 (선택, 사용자 톤에 맞춰)
4. 후보가 여럿이면 사용자에게 확인

## Workflow: 노트 작성

"X 메모해 / X 정리해 / X 적어둬" (시간 로그 아닌 경우):

1. 위치 결정 (Folder map 참고)
2. 파일명 결정 (자연어 제목, 기존 노트와 중복되면 묻기)
3. `_templates/note.md` 또는 `_templates/project.md` 베이스로 frontmatter 작성
4. 본문 작성. 관련 노트 `[[wikilink]]` 로 연결.

## Workflow: 어제 / 이번 주 회고

- "어제 뭐 했지?" → `daily/<YESTERDAY>.md` 읽고 요약
- "이번 주 한 일" → `daily/` 에서 최근 7일 파일 읽고 합쳐 요약

## 절대 안 함

- `<% tp.* %>` Templater 코드를 파일에 그대로 쓰기 (반드시 평가)
- 사용자 동의 없이 `archive/` 로 옮기기
- 사용자 동의 없이 파일 삭제
- 시간 추측해서 적기 (반드시 `date` 호출)
- 옵시디언 외 경로에 옵시디언 노트 만들기

## 시작 체크

작업 들어가기 전 한 번 확인:
- 볼트 경로 존재? (없으면 사용자에게 묻기)
- 폴더 구조 일치? (daily/projects/notes/archive/_templates 5개)

불일치 시 사용자에게 알리고 진행 멈춤.
