# claude-obsidian

Claude Code skills and commands for Obsidian vault management.

전제: 옵시디언은 **읽기 전용 뷰어**, 모든 작성/편집은 Claude 경유.

## 구성

```
.claude/
├── skills/obsidian/SKILL.md   자연어 트리거 스킬
├── commands/
│   ├── daily.md               /daily — 오늘 데일리 보장 + carry-over
│   ├── note.md                /note <주제> — 새 노트 생성
│   ├── vault-sync.md          /vault-sync — git commit/push
│   └── vault-status.md        /vault-status — 볼트 현황 한눈에
└── settings.json              볼트 경로 권한
```

레포가 통째로 `.claude/` 하위 구조라서:
- 글로벌 사용: `~/.claude/skills`, `~/.claude/commands` 로 심링크
- 프로젝트 한정: 어떤 프로젝트에 `.claude/`로 클론하면 그 안에서만 적용
- 이 레포 자체에서 `claude` 켜도 자동 로드 (자기 자신을 사용)

## 볼트 구조

```
daily/        날짜별 로그 (YYYY-MM-DD.md)
projects/     끝 시점 있는 큰 작업
notes/        그 외 다 (메모 / 지식 / 참고)
archive/      안 보는 것
_templates/   Templater 템플릿 (daily / project / note)
```

## 데일리 노트 패턴

타임라인 + Tasks. AI가 시간 자동 기록.

```markdown
# 2026-04-28 (화)

## 로그
- 09:30 김부장 회의 — Q2 발표자료
- 10:15 [[발표자료]] 초안 시작

## Tasks
- [ ] 발표자료 1차 완성
- [x] 메일 답장 ✅ 09:00

## Notes

## Tomorrow
- [ ]
```

## 설치 (새 머신에서)

```bash
# 1. 클론
git clone git@github.com:wooinwoo/claude-obsidian.git ~/projects/personal/claude-obsidian

# 2. 글로벌 심링크
mkdir -p ~/.claude/skills ~/.claude/commands
ln -s ~/projects/personal/claude-obsidian/.claude/skills/obsidian ~/.claude/skills/obsidian
for f in ~/projects/personal/claude-obsidian/.claude/commands/*.md; do
  ln -s "$f" ~/.claude/commands/"$(basename "$f")"
done

# 3. settings.json 권한 병합
#    ~/.claude/settings.json 의 permissions.allow 에
#    레포의 .claude/settings.json 내용을 병합 (볼트 경로는 머신에 맞게 수정)
```

## 스킬 트리거

자연어로 트리거 (스킬 description이 자동 매칭):
- "옵시디언에 X 메모해", "볼트에 X 적어둬"
- "오늘 데일리에 X", "지금 X 했어 (적어둬)"
- "X 끝냈어" → 해당 태스크 토글
- "어제 뭐 했지?" → 어제 데일리 요약

명시적 호출:
- `/daily` `/note <주제>` `/vault-sync` `/vault-status`

## 업데이트

```bash
cd ~/projects/personal/claude-obsidian && git pull
```

심볼릭 링크라서 즉시 반영.

## 커스터마이징

- 볼트 경로가 다른 머신: `.claude/skills/obsidian/SKILL.md` 와 `.claude/settings.json` 의 경로를 수정.
- 새 슬래시 커맨드 추가: `.claude/commands/<이름>.md` 작성, 글로벌 심링크 추가.
- 스킬 동작 수정: `.claude/skills/obsidian/SKILL.md` 편집 (심링크라 즉시 반영).
