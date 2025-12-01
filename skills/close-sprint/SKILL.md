---
name: close-sprint
description: |
  Sprint 종료 및 회고 정리. Use when (1) Sprint 마감,
  (2) 회고 작성, (3) /SAX:sprint close 커맨드.
tools: [Bash, Read, Write]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: close-sprint 호출` 메시지를 첫 줄에 출력하세요.

# close-sprint Skill

> Sprint 종료 처리 및 회고 생성

## Purpose

Sprint를 종료하고 회고를 정리하며, 미완료 Task를 다음 Sprint로 이관합니다.

## Workflow

```
Sprint 종료 요청
    ↓
1. 완료/미완료 Task 집계
2. Velocity 계산
3. 회고 요약 생성
4. Sprint Issue 업데이트
5. Milestone 종료
6. 미완료 Task → 다음 Sprint 이관
7. sprint-current 라벨 제거
    ↓
완료
```

## Input

```yaml
sprint_name: "Sprint 23"          # 필수
next_sprint: "Sprint 24"          # 선택 (미완료 이관용)
retrospective:                    # 선택
  good:
    - "API 개발 순조로움"
  improve:
    - "테스트 커버리지 부족"
```

## Output

```markdown
[SAX] Skill: close-sprint 완료

✅ Sprint 23 종료 완료

**완료**: 8/10 Task (80%)
**Velocity**: 32pt
**미완료 이관**: 2 Task → Sprint 24
```

## API 호출

### Sprint Task 집계

```bash
# Sprint 23 Task 조회
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --state all \
  --json number,title,state,labels,assignees
```

### Velocity 계산

```bash
# 완료된 Task의 Point 합계
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --state closed \
  --json labels \
  | jq '[.[] | .labels[] | select(.name | startswith("point-")) | .name | split("-")[1] | tonumber] | add'
```

### Milestone 종료

```bash
# Milestone 번호 조회
MILESTONE_NUMBER=$(gh api repos/semicolon-devteam/docs/milestones \
  --jq '.[] | select(.title == "Sprint 23") | .number')

# Milestone 종료
gh api repos/semicolon-devteam/docs/milestones/$MILESTONE_NUMBER \
  -X PATCH \
  -f state="closed"
```

### 미완료 Task 이관

```bash
# 미완료 Task → 다음 Sprint
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --state open \
  --json number \
  | jq -r '.[].number' \
  | xargs -I {} gh issue edit {} \
    --remove-label "sprint-23" \
    --add-label "sprint-24" \
    --milestone "Sprint 24"
```

### Sprint Issue 업데이트

```bash
# 회고 추가
gh issue comment {sprint_issue_number} \
  --repo semicolon-devteam/docs \
  --body "$(cat <<'EOF'
## 📝 Sprint 회고

### ✅ 잘된 점
- API 개발 순조로움
- 팀 협업 원활

### 🔧 개선할 점
- 테스트 커버리지 부족
- 코드 리뷰 지연

### 📊 통계
- 완료: 8/10 Task (80%)
- Velocity: 32pt
- 미완료 이관: 2 Task → Sprint 24
EOF
)"

# sprint-current 라벨 제거
gh issue edit {sprint_issue_number} \
  --repo semicolon-devteam/docs \
  --remove-label "sprint-current" \
  --add-label "sprint-closed"
```

## 회고 템플릿

```markdown
## 📝 Sprint 회고

### ✅ 잘된 점
{good_points}

### 🔧 개선할 점
{improve_points}

### 📊 통계
- **완료**: {done_count}/{total_count} Task ({completion_rate}%)
- **Velocity**: {velocity}pt
- **미완료 이관**: {carry_over_count} Task → {next_sprint}

### 📈 트렌드
| Sprint | Velocity | 완료율 |
|--------|----------|--------|
| {prev_sprint_2} | {prev_velocity_2}pt | {prev_rate_2}% |
| {prev_sprint_1} | {prev_velocity_1}pt | {prev_rate_1}% |
| {current_sprint} | {velocity}pt | {completion_rate}% |
```

## 완료 메시지

```markdown
[SAX] Skill: close-sprint 완료

✅ **{sprint_name}** 종료 완료

## 📊 Sprint 결과

| 항목 | 값 |
|------|-----|
| 완료 Task | {done_count}/{total_count} ({completion_rate}%) |
| Velocity | {velocity}pt |
| 미완료 이관 | {carry_over_count} Task → {next_sprint} |

## 📈 Velocity 트렌드
| Sprint | Velocity |
|--------|----------|
| {sprint_name} | {velocity}pt |
| 3 Sprint 평균 | {avg_velocity}pt |

{next_sprint}이 준비되었습니다.
```
