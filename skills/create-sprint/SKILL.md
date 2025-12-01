---
name: create-sprint
description: |
  Sprint Issue 및 Milestone 생성. Use when (1) 새 Sprint 시작,
  (2) Sprint 계획 수립, (3) /SAX:sprint create 커맨드.
tools: [Bash, Read, Write]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: create-sprint 호출` 메시지를 첫 줄에 출력하세요.

# create-sprint Skill

> Sprint Issue 및 GitHub Milestone 생성

## Purpose

새로운 Sprint를 생성하고 관련 GitHub 리소스를 설정합니다.

## Workflow

```
Sprint 생성 요청
    ↓
1. Sprint 정보 수집 (이름, 기간, 목표)
2. GitHub Milestone 생성
3. Sprint Issue 생성 (docs 레포)
4. Projects #1 연결
5. sprint-current 라벨 관리
    ↓
완료
```

## Input

```yaml
sprint_name: "Sprint 23"          # 필수
start_date: "2024-12-02"          # 필수
end_date: "2024-12-13"            # 필수
goals:                            # 선택
  - "댓글 기능 완성"
  - "알림 연동 시작"
```

## Output

```markdown
[SAX] Skill: create-sprint 완료

✅ Sprint 23 생성 완료

**Milestone**: [Sprint 23](milestone_url)
**Sprint Issue**: [#123](issue_url)
**기간**: 2024-12-02 ~ 2024-12-13
```

## API 호출

### Milestone 생성

```bash
gh api repos/semicolon-devteam/docs/milestones \
  -X POST \
  -f title="Sprint 23" \
  -f due_on="2024-12-13T23:59:59Z" \
  -f description="Sprint 23: 댓글 기능 완성"
```

### Sprint Issue 생성

```bash
gh issue create \
  --repo semicolon-devteam/docs \
  --title "🏃 Sprint 23: 댓글 기능 완성" \
  --label "sprint,sprint-current" \
  --milestone "Sprint 23" \
  --body "$(cat <<'EOF'
# 🏃 Sprint 23: 댓글 기능 완성

**기간**: 2024-12-02 ~ 2024-12-13
**Milestone**: Sprint 23

## 🎯 Sprint 목표
- 댓글 기능 완성
- 알림 연동 시작

## 📋 포함된 Task
| # | Task | Point | 담당자 | 상태 |
|---|------|-------|--------|------|
| - | (할당 예정) | - | - | - |

## 📊 용량
- **총 Point**: 0
- **팀 용량**: 40pt
- **여유**: 40pt

## 📈 진행 상황
- ✅ 완료: 0 (0pt)
- 🔄 진행중: 0 (0pt)
- ⏳ 대기: 0 (0pt)
EOF
)"
```

### 이전 Sprint 정리

```bash
# 이전 sprint-current 라벨 제거
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-current" \
  --json number \
  | jq -r '.[].number' \
  | xargs -I {} gh issue edit {} --remove-label "sprint-current"
```

## Sprint Issue 템플릿

```markdown
# 🏃 {sprint_name}: {sprint_goal}

**기간**: {start_date} ~ {end_date}
**Milestone**: [{sprint_name}]({milestone_url})

## 🎯 Sprint 목표
{goals_list}

## 📋 포함된 Task
| # | Task | Point | 담당자 | 상태 |
|---|------|-------|--------|------|
{task_rows}

## 📊 용량
- **총 Point**: {total_points}
- **팀 용량**: {capacity}pt
- **여유**: {remaining}pt

## 📈 진행 상황
- ✅ 완료: {done_count} ({done_points}pt)
- 🔄 진행중: {progress_count} ({progress_points}pt)
- ⏳ 대기: {todo_count} ({todo_points}pt)
```

## 완료 메시지

```markdown
[SAX] Skill: create-sprint 완료

✅ **{sprint_name}** 생성 완료

| 항목 | 값 |
|------|-----|
| Milestone | [{sprint_name}]({milestone_url}) |
| Sprint Issue | [#{issue_number}]({issue_url}) |
| 기간 | {start_date} ~ {end_date} |
| 팀 용량 | {capacity}pt |

다음 단계: `/SAX:sprint add` 명령어로 Task를 Sprint에 할당하세요.
```
