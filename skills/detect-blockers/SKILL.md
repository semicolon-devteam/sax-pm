---
name: detect-blockers
description: |
  블로커 및 지연 Task 감지. Use when (1) 블로커 확인,
  (2) 지연 현황 조회, (3) 자동 모니터링.
tools: [Bash, Read]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: detect-blockers 호출` 메시지를 첫 줄에 출력하세요.

# detect-blockers Skill

> 블로커 및 지연 Task 감지

## Purpose

프로젝트 진행을 방해하는 블로커와 지연된 Task를 자동으로 감지합니다.

## Workflow

```
블로커 감지 요청
    ↓
1. In Progress 장기 Task 감지 (3일+)
2. blocked 라벨 Task 조회
3. 의존성 미해결 Task 확인
4. 심각도 분류
5. 알림 (Critical 시)
    ↓
완료
```

## Input

```yaml
sprint_name: "Sprint 23"          # 선택 (기본: 현재 Sprint)
threshold_days: 3                 # 선택 (지연 판정 기준, 기본 3일)
notify: true                      # 선택 (Slack 알림 여부)
```

## Output

```markdown
# 🚨 블로커 현황

**기준일**: 2024-12-10
**Sprint**: Sprint 23

## 🔴 Critical (즉시 조치 필요)

| # | Task | 담당자 | 지연 | 원인 |
|---|------|--------|------|------|
| #234 | 댓글 API | @kyago | 5일 | blocked 라벨 |

## 🟡 Warning (주의 필요)

| # | Task | 담당자 | 지연 | 원인 |
|---|------|--------|------|------|
| #456 | 알림 연동 | @Garden | 3일 | In Progress 장기 |

## 📊 요약
- Critical: 1
- Warning: 1
- 총 블로커: 2
```

## 감지 규칙

### 지연 판정

| 상태 | 경과 시간 | 심각도 |
|------|----------|--------|
| In Progress | 3-4일 | 🟡 Warning |
| In Progress | 5일+ | 🔴 Critical |
| Review | 2일+ | 🟡 Warning |
| Review | 4일+ | 🔴 Critical |

### 블로커 유형

| 유형 | 감지 방법 | 심각도 |
|------|----------|--------|
| **장기 지연** | 상태 경과 시간 | 경과에 따라 |
| **명시적 차단** | blocked 라벨 | 🔴 Critical |
| **의존성 미해결** | 연결된 Issue 미완료 | 🟡 Warning |
| **담당자 미할당** | assignee 없음 | 🟡 Warning |

## API 호출

### In Progress Task 조회

```bash
# In Progress 상태 Task (Projects 기준)
gh api graphql -f query='
{
  repository(owner: "semicolon-devteam", name: "docs") {
    issues(first: 100, labels: ["sprint-23"], states: [OPEN]) {
      nodes {
        number
        title
        createdAt
        updatedAt
        assignees(first: 3) { nodes { login } }
        projectItems(first: 1) {
          nodes {
            fieldValues(first: 10) {
              nodes {
                ... on ProjectV2ItemFieldSingleSelectValue {
                  name
                  updatedAt
                }
              }
            }
          }
        }
      }
    }
  }
}'
```

### blocked 라벨 Task

```bash
gh issue list \
  --repo semicolon-devteam/docs \
  --label "blocked" \
  --state open \
  --json number,title,assignees,updatedAt
```

## 지연 일수 계산

```javascript
function calculateDelayDays(lastUpdate) {
  const now = new Date();
  const updated = new Date(lastUpdate);
  const diffMs = now - updated;
  const diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
  return diffDays;
}
```

## Slack 알림

Critical 블로커 발견 시 자동 알림:

```bash
# notify-slack 호출
/SAX:slack #_협업 채널에 블로커 알림
```

**메시지 형식**:
```
🚨 *블로커 감지*

Sprint 23에서 Critical 블로커가 발견되었습니다.

• #234 댓글 API (@kyago) - 5일 지연

즉시 확인이 필요합니다.
```

## 완료 메시지

```markdown
[SAX] Skill: detect-blockers 완료

# 🚨 블로커 현황

**기준일**: {report_date}
**Sprint**: {sprint_name}

## 🔴 Critical ({critical_count})
{critical_table}

## 🟡 Warning ({warning_count})
{warning_table}

## 📊 요약
- Critical: {critical_count}
- Warning: {warning_count}
- 총 블로커: {total_count}

{slack_notification_status}
```
