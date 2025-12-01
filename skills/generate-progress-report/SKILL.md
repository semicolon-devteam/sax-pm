---
name: generate-progress-report
description: |
  Sprint 진행도 리포트 생성. Use when (1) 진행 현황 조회,
  (2) /SAX:progress 커맨드, (3) 상태 리포트 요청.
tools: [Bash, Read, Write]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: generate-progress-report 호출` 메시지를 첫 줄에 출력하세요.

# generate-progress-report Skill

> Sprint 진행도 리포트 생성

## Purpose

현재 Sprint의 진행 상황을 분석하고 리포트를 생성합니다.

## Workflow

```
진행도 리포트 요청
    ↓
1. 현재 Sprint 식별
2. Task 상태별 집계
3. 담당자별 현황 집계
4. 진행률 계산
5. 리포트 생성
    ↓
완료
```

## Input

```yaml
sprint_name: "Sprint 23"          # 선택 (기본: 현재 Sprint)
format: "markdown"                # 선택 (markdown|slack)
```

## Output

```markdown
# 📊 Sprint 23 진행 현황

**기간**: 2024-12-02 ~ 2024-12-13
**진행률**: ████████░░ 78%

## 📈 상태별 현황
| 상태 | 개수 | Point |
|------|------|-------|
| ✅ Done | 7 | 21 |
| 🔄 In Progress | 3 | 9 |
| ⏳ Todo | 2 | 6 |

## 👥 담당자별 현황
| 담당자 | 완료 | 진행중 | 대기 | 완료율 |
|--------|------|--------|------|--------|
| @kyago | 3 | 1 | 0 | 75% |
| @Garden | 2 | 1 | 1 | 50% |
| @Roki | 2 | 1 | 1 | 50% |

## ⏱️ 일정 현황
- **남은 기간**: D-3
- **예상 완료율**: 90%
```

## API 호출

### 현재 Sprint 조회

```bash
# sprint-current 라벨 Issue
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint,sprint-current" \
  --json number,title \
  --jq '.[0]'
```

### Task 상태별 집계

```bash
# Sprint Task 전체 조회
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --state all \
  --json number,title,state,labels,assignees
```

### Projects 상태 조회

```bash
# GitHub Projects에서 상태 조회
gh api graphql -f query='
{
  repository(owner: "semicolon-devteam", name: "docs") {
    issues(first: 100, labels: ["sprint-23"]) {
      nodes {
        number
        title
        state
        assignees(first: 3) {
          nodes { login }
        }
        projectItems(first: 1) {
          nodes {
            fieldValues(first: 10) {
              nodes {
                ... on ProjectV2ItemFieldSingleSelectValue {
                  name
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

## Progress Bar 생성

```javascript
function generateProgressBar(percent) {
  const filled = Math.round(percent / 10);
  const empty = 10 - filled;
  return '█'.repeat(filled) + '░'.repeat(empty) + ` ${percent}%`;
}

// 78% → ████████░░ 78%
```

## 상태 매핑

| Projects 상태 | 표시 | 아이콘 |
|--------------|------|--------|
| Todo | 대기 | ⏳ |
| In Progress | 진행중 | 🔄 |
| Review | 리뷰 | 👀 |
| Done | 완료 | ✅ |

## 완료 메시지

```markdown
[SAX] Skill: generate-progress-report 완료

# 📊 {sprint_name} 진행 현황

**기간**: {start_date} ~ {end_date}
**진행률**: {progress_bar}

## 📈 상태별 현황
| 상태 | 개수 | Point |
|------|------|-------|
| ✅ Done | {done_count} | {done_points} |
| 🔄 In Progress | {progress_count} | {progress_points} |
| ⏳ Todo | {todo_count} | {todo_points} |

## 👥 담당자별 현황
| 담당자 | 완료 | 진행중 | 대기 | 완료율 |
|--------|------|--------|------|--------|
{member_rows}

## ⏱️ 일정 현황
- **남은 기간**: D-{days_remaining}
- **예상 완료율**: {estimated_completion}%

{blockers_warning}
```
