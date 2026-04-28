---
description: 옵시디언 볼트에 새 노트를 만든다
argument-hint: <주제 또는 내용>
---

볼트에 새 노트를 만든다. 인자(`$ARGUMENTS`)는 노트의 주제 또는 첫 줄 내용.

1. **위치 결정** (`obsidian` 스킬의 Folder map 참고)
   - 끝나는 시점 있는 큰 작업 같으면 → `projects/`
   - 그 외 다 → `notes/`
   - 모호하면 사용자에게 한 번 묻기 (선택지 2개만 제시).

2. **파일명 결정**
   - 인자에서 자연어 제목 뽑기 (한글/공백 OK).
   - 기존 파일과 중복되면 사용자에게 묻기 (덮어쓸지 / 다른 이름).

3. **템플릿 선택**
   - `projects/` → `_templates/project.md` 베이스
   - `notes/` → `_templates/note.md` 베이스
   - Templater 변수는 모두 평가 후 작성 (`<% tp.date.now() %>` 그대로 쓰지 말 것).

4. **frontmatter 채우기**
   - `created`: 오늘 날짜
   - `tags`: 인자에서 추론 가능하면 추가 (예: 인자가 "Docker 컨테이너 노트" → `tags: [note, docker]`). 추론 안 되면 `[note]` 또는 `[project]` 만.

5. **본문 작성**
   - 인자가 단순 주제(짧은 단어)면 빈 본문 + H1만.
   - 인자가 문장/내용이면 본문에 정리해서 작성.

6. **결과 보고**: 파일 경로 1줄, 주요 frontmatter 1줄.
