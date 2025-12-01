---
name: calculate-velocity
description: |
  팀 Velocity 계산. Use when (1) Sprint 종료 시 Velocity 기록,
  (2) 생산성 분석, (3) 일정 예측 시.
tools: [Bash, Read]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: calculate-velocity 호출` 메시지를 첫 줄에 출력하세요.

# calculate-velocity Skill

> 팀 Velocity 계산 및 트렌드 분석

## Purpose

최근 Sprint들의 완료 Point를 기반으로 팀 Velocity를 계산합니다.

## Workflow

```
Velocity 계산 요청
    ↓
1. 최근 N Sprint 조회 (기본 3)
2. Sprint별 완료 Point 집계
3. 평균 Velocity 계산
4. 트렌드 분석
    ↓
완료
```

## Input

```yaml
sprint_count: 3                   # 선택 (기본 3)
include_current: false            # 선택 (진행중 Sprint 포함 여부)
```

## Output

```markdown
[SAX] Skill: calculate-velocity 완료

📊 팀 Velocity 분석

**평균 Velocity**: 35pt/Sprint

| Sprint | 완료 Point | 목표 | 달성률 |
|--------|-----------|------|--------|
| Sprint 21 | 35pt | 40pt | 88% |
| Sprint 22 | 38pt | 40pt | 95% |
| Sprint 23 | 32pt | 40pt | 80% |

**트렌드**: ↘️ 소폭 하락 (-8%)
```

## API 호출

### Sprint 목록 조회

```bash
# 종료된 Sprint Milestone 조회
gh api repos/semicolon-devteam/docs/milestones \
  -q '.[] | select(.state == "closed") | {title, closed_at}' \
  | jq -s 'sort_by(.closed_at) | reverse | .[0:3]'
```

### Sprint별 완료 Point

```bash
# Sprint N의 완료 Point 합계
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-{N}" \
  --state closed \
  --json labels \
  | jq '[.[] | .labels[] | select(.name | startswith("point-")) | .name | split("-")[1] | tonumber] | add // 0'
```

## Velocity 계산

```javascript
function calculateVelocity(sprints) {
  const velocities = sprints.map(s => s.completedPoints);
  const sum = velocities.reduce((a, b) => a + b, 0);
  return sum / velocities.length;
}

// 예:
// Sprint 21: 35pt
// Sprint 22: 38pt
// Sprint 23: 32pt
// 평균: (35 + 38 + 32) / 3 = 35pt
```

## 트렌드 분석

```javascript
function analyzeTrend(velocities) {
  const latest = velocities[0];
  const previous = velocities[1];
  const diff = ((latest - previous) / previous) * 100;

  if (diff > 10) return { icon: '📈', text: '상승', diff };
  if (diff > 0) return { icon: '↗️', text: '소폭 상승', diff };
  if (diff > -10) return { icon: '↘️', text: '소폭 하락', diff };
  return { icon: '📉', text: '하락', diff };
}
```

## 일정 예측

Velocity를 활용한 일정 예측:

```javascript
function predictCompletion(remainingPoints, velocity) {
  const sprintsNeeded = remainingPoints / velocity;
  const daysNeeded = sprintsNeeded * 10; // 2주 = 10 영업일

  return {
    sprints: Math.ceil(sprintsNeeded),
    days: Math.ceil(daysNeeded),
    estimatedDate: addBusinessDays(new Date(), daysNeeded)
  };
}

// 예:
// 남은 Point: 70pt
// Velocity: 35pt/Sprint
// 예상: 2 Sprint (20일)
```

## 완료 메시지

```markdown
[SAX] Skill: calculate-velocity 완료

## 📊 팀 Velocity 분석

**평균 Velocity**: {avg_velocity}pt/Sprint

### Sprint별 실적

| Sprint | 완료 | 목표 | 달성률 |
|--------|------|------|--------|
{sprint_rows}

### 트렌드
{trend_icon} **{trend_text}** ({trend_diff:+.1f}%)

### 예측
현재 Velocity 기준:
- 10pt 작업: ~3일
- 20pt 작업: ~6일
- 40pt 작업: ~1 Sprint (2주)
```
