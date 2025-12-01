---
name: generate-member-report
description: |
  인원별 업무 현황 리포트 생성. Use when (1) 담당자별 현황 조회,
  (2) /SAX:report member 커맨드, (3) 업무량 분석.
tools: [Bash, Read, Write]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: generate-member-report 호출` 메시지를 첫 줄에 출력하세요.

# generate-member-report Skill

> 인원별 업무 현황 리포트 생성

## Purpose

팀원별 Task 할당 및 진행 현황을 분석하고 리포트를 생성합니다.

## Workflow

```
인원별 리포트 요청
    ↓
1. 대상 인원 확인 (특정/전체)
2. 인원별 Task 그룹화
3. 완료율/업무량 계산
4. 리포트 생성
    ↓
완료
```

## Input

```yaml
member: "@kyago"                  # 선택 (기본: 전체)
sprint_name: "Sprint 23"          # 선택 (기본: 현재 Sprint)
format: "markdown"                # 선택
```

## Output (전체)

```markdown
# 👥 팀원별 업무 현황

**Sprint**: Sprint 23
**기간**: 2024-12-02 ~ 2024-12-13

## 📊 요약

| 담당자 | 할당 | 완료 | 진행중 | 대기 | 완료율 |
|--------|------|------|--------|------|--------|
| @kyago | 12pt | 8pt | 3pt | 1pt | 67% |
| @Garden | 10pt | 7pt | 3pt | 0pt | 70% |
| @Roki | 8pt | 6pt | 2pt | 0pt | 75% |

## 🔥 주요 현황

**가장 높은 완료율**: @Roki (75%)
**가장 많은 할당**: @kyago (12pt)
**블로커 보유**: @kyago (#234)
```

## Output (개인)

```markdown
# 👤 @kyago 업무 현황

**Sprint**: Sprint 23
**기간**: 2024-12-02 ~ 2024-12-13

## 📊 요약

| 항목 | 값 |
|------|-----|
| 할당 Point | 12pt |
| 완료 Point | 8pt |
| 완료율 | 67% |
| 용량 대비 | 120% ⚠️ |

## ✅ 완료 (3)
- [x] #450 로그인 페이지 리팩토링 (3pt)
- [x] #451 에러 핸들링 개선 (2pt)
- [x] #452 테스트 코드 작성 (3pt)

## 🔄 진행중 (1)
- [ ] #456 댓글 API 구현 (3pt) - 70% 완료

## ⏳ 대기 (1)
- [ ] #458 알림 연동 (1pt)

## ⚠️ 블로커
- #234: 의존성 미해결 (3일 지연)
```

## API 호출

### 인원별 Task 조회

```bash
# 특정 담당자의 Sprint Task
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --assignee "kyago" \
  --state all \
  --json number,title,state,labels
```

### 전체 팀원 Task

```bash
# Sprint Task 전체 조회 후 담당자별 그룹화
gh issue list \
  --repo semicolon-devteam/docs \
  --label "sprint-23" \
  --state all \
  --json number,title,state,labels,assignees
```

## 업무량 분석

### 용량 대비 할당률

```javascript
function calculateCapacityUsage(assigned, capacity) {
  const usage = (assigned / capacity) * 100;

  if (usage <= 80) return { status: '🟢', text: '적정' };
  if (usage <= 100) return { status: '🟡', text: '주의' };
  return { status: '🔴', text: '초과' };
}
```

### 완료율 계산

```javascript
function calculateCompletionRate(done, total) {
  if (total === 0) return 0;
  return Math.round((done / total) * 100);
}
```

## Semicolon 팀원 목록

| GitHub ID | 이름 | 기본 용량 |
|-----------|------|----------|
| kyago | 강용준 | 10pt |
| garden92 | 서정원 | 10pt |
| Roki-Noh | 노영록 | 10pt |
| beomsun1234 | 장현봉 | 10pt |
| DwightKang | 강동현 | 10pt |
| yeomso | 염현준 | 10pt |
| reus-jeon | 전준영 | 7pt |

## 완료 메시지

```markdown
[SAX] Skill: generate-member-report 완료

# 👥 팀원별 업무 현황

**Sprint**: {sprint_name}

## 📊 요약

| 담당자 | 할당 | 완료 | 완료율 | 상태 |
|--------|------|------|--------|------|
{member_rows}

## 🔥 주요 현황

- **가장 높은 완료율**: {top_performer}
- **업무 과중**: {overloaded_members}
- **블로커 보유**: {blocked_members}
```
