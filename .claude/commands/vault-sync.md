---
description: 옵시디언 볼트를 git commit/push 한다
---

옵시디언 볼트의 변경사항을 커밋하고 푸시한다.

1. 볼트 디렉토리: `/mnt/c/Users/rst010/Documents/Obsidian Vault`
2. `git status --short` 로 변경사항 확인.
   - 변경 없음: "변경사항 없음" 알리고 종료.
3. `git add -A`
4. 커밋 메시지: `vault: <YYYY-MM-DD HH:MM>` (현재 날짜+시간, `date '+%Y-%m-%d %H:%M'`)
   - 변경 파일이 1~2개면 메시지 뒤에 ` - <파일명>` 추가.
5. `git commit -m "<message>"`
6. `git push` (실패하면 에러 그대로 사용자에게 보임 — pull 필요할 수 있음).
7. 결과 보고: 커밋 해시 짧게 + 변경 파일 수.

주의:
- 절대 `--force` 푸시 안 함.
- 시크릿 의심 파일(`.env`, `credentials*`) 있으면 멈추고 사용자에게 알림.
