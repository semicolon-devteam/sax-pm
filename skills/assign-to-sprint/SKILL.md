---
name: assign-to-sprint
description: |
  Task를 Sprint에 할당. Use when (1) Sprint 계획 시 Task 선정,
  (2) Task 추가 할당, (3) /SAX:sprint add 커맨드.
tools: [Bash, Read]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: assign-to-sprint 호출` 메시지를 첫 줄에 출력하세요.

# assign-to-sprint Skill

> Task를 Sprint에 할당하고 라벨/마일스톤 설정

## Purpose

Backlog의 Task를 특정 Sprint에 할당합니다.

## Workflow

```
Task 할당 요청
    ↓
1. 대상 Task 확인
2. 용량 체크 (초과 경고)
3. sprint-N 라벨 추가
4. Milestone 연결
5. sprint-backlog 라벨 제거
6. Sprint Issue 업데이트
    ↓
완료
```

## Input

```yaml
sprint_name: "Sprint 23"          # 필수
task_numbers:                     # 필수
  - 123
  - 124
  - 125
```

## Output

```markdown
[SAX] Skill: assign-to-sprint 완료

✅ 3개 Task를 Sprint 23에 할당했습니다.

| # | Task | Point | 담당자 |
|---|------|-------|--------|
| #123 | 댓글 API | 5 | @kyago |
| #124 | 댓글 UI | 3 | @Garden |
| #125 | 알림 연동 | 5 | @Roki |

**Sprint 용량**: 35/40pt (87%)
```

## API 호출

### Task 정보 조회

```bash
# Task 상세 조회
gh issue view {number} \
  --repo semicolon-devteam/docs \
  --json number,title,labels,assignees
```

### 라벨/마일스톤 설정

```bash
# Sprint 라벨 추가, backlog 라벨 제거
gh issue edit {number} \
  --repo semicolon-devteam/docs \
  --add-label "sprint-23" \
  --remove-label "sprint-backlog" \
  --milestone "Sprint 23"
```

### 용량 체크

```bash
# 현재 Sprint 할당량 조회
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --json labels \
  | jq '[.[] | .labels[] | select(.name | startswith("point-")) | .name | split("-")[1] | tonumber] | add'
```

## 용량 경고

### 정상 (80% 미만)

```markdown
✅ 3개 Task 할당 완료

**Sprint 용량**: 28/40pt (70%)
```

### 주의 (80-90%)

```markdown
⚠️ 3개 Task 할당 완료

**Sprint 용량**: 35/40pt (87%) - 주의

Sprint 용량이 87%입니다. 추가 할당 시 주의하세요.
```

### 위험 (90% 이상)

```markdown
🚨 용량 초과 경고

현재 Sprint 할당량: 42/40pt (105%)

**권장 조치**:
1. 우선순위 낮은 Task 다음 Sprint로 이관
2. Task 분할
3. 리소스 추가 검토

그래도 할당하시겠습니까? (y/n)
```

## Sprint Issue 업데이트

할당 후 Sprint Issue의 Task 테이블 업데이트:

```markdown
## 📋 포함된 Task
| # | Task | Point | 담당자 | 상태 |
|---|------|-------|--------|------|
| #123 | 댓글 API | 5 | @kyago | ⏳ |
| #124 | 댓글 UI | 3 | @Garden | ⏳ |
| #125 | 알림 연동 | 5 | @Roki | ⏳ |

## 📊 용량
- **총 Point**: 13
- **팀 용량**: 40pt
- **여유**: 27pt
```

## 완료 메시지

```markdown
[SAX] Skill: assign-to-sprint 완료

✅ {count}개 Task를 {sprint_name}에 할당했습니다.

| # | Task | Point | 담당자 |
|---|------|-------|--------|
{task_rows}

**Sprint 용량**: {assigned}/{capacity}pt ({usage}%)
**Sprint Issue**: [#{sprint_issue}]({issue_url}) 업데이트됨
```
