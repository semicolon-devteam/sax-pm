# Sprint Workflow

> Sprint 생성부터 종료까지의 전체 워크플로우

## Sprint 생명주기

```
[Planning] → [Active] → [Review] → [Closed]
    ↓           ↓          ↓          ↓
  Task 선정   Daily 추적  회고 작성   Velocity 기록
```

## Phase 1: Planning (계획)

### 1.1 Sprint 생성

```bash
# GitHub Milestone 생성
gh api repos/semicolon-devteam/docs/milestones \
  -X POST \
  -f title="Sprint 23" \
  -f due_on="2024-12-13T23:59:59Z" \
  -f description="Sprint 23 목표: 댓글 기능 완성"
```

### 1.2 Backlog 조회

```bash
# sprint-backlog 라벨 Task 조회
gh issue list \
  --repo semicolon-devteam/docs \
  --label "task,sprint-backlog" \
  --json number,title,labels,assignees
```

### 1.3 Task 선정

**선정 기준**:
1. 우선순위 (high-priority 우선)
2. 의존성 (선행 Task 완료 여부)
3. 팀 용량 (초과 방지)

### 1.4 Task 할당

```bash
# sprint-N 라벨 추가
gh issue edit {number} \
  --repo semicolon-devteam/docs \
  --add-label "sprint-23" \
  --remove-label "sprint-backlog" \
  --milestone "Sprint 23"
```

---

## Phase 2: Active (실행)

### 2.1 Daily 추적

매일 확인 사항:
- [ ] 진행중 Task 상태 확인
- [ ] 블로커 감지
- [ ] 예상 지연 파악

### 2.2 상태 업데이트

Projects #1에서 Task 상태 관리:
- `Todo` → `In Progress` → `Done`

### 2.3 블로커 대응

블로커 발견 시:
1. `blocked` 라벨 추가
2. 블로킹 이슈 연결
3. Slack 알림

---

## Phase 3: Review (검토)

### 3.1 Sprint 종료일

Sprint 종료일(금요일) 오후:
1. 미완료 Task 확인
2. 다음 Sprint 이관 결정
3. 회고 준비

### 3.2 회고 작성

Sprint Issue에 회고 추가:

```markdown
## 📝 Sprint 회고

### ✅ 잘된 점
- {good_1}
- {good_2}

### 🔧 개선할 점
- {improve_1}
- {improve_2}

### 📊 통계
- 완료: {done_count}/{total_count} ({completion_rate}%)
- Velocity: {velocity}pt
```

---

## Phase 4: Closed (종료)

### 4.1 Milestone 종료

```bash
gh api repos/semicolon-devteam/docs/milestones/{milestone_number} \
  -X PATCH \
  -f state="closed"
```

### 4.2 미완료 Task 이관

```bash
# 다음 Sprint로 라벨 변경
gh issue edit {number} \
  --repo semicolon-devteam/docs \
  --remove-label "sprint-23" \
  --add-label "sprint-24" \
  --milestone "Sprint 24"
```

### 4.3 Velocity 기록

Sprint Issue에 최종 Velocity 기록:

```markdown
## 📈 최종 Velocity

| Sprint | 완료 Point | 목표 Point | 달성률 |
|--------|-----------|-----------|--------|
| Sprint 23 | 32 | 40 | 80% |
| Sprint 22 | 38 | 40 | 95% |
| Sprint 21 | 35 | 40 | 88% |

**평균 Velocity**: 35pt/Sprint
```

---

## Sprint 간 전환

```
Sprint N 종료
    ↓
미완료 Task → Sprint N+1 이관
    ↓
Sprint N Milestone 종료
    ↓
Sprint N+1 계획 시작
```

## 예외 상황

### 긴급 Task 추가

Sprint 중간에 긴급 Task 추가 시:
1. 용량 재계산
2. 기존 Task 우선순위 조정
3. 필요시 Task 이관

### Sprint 취소

Sprint 취소 시:
1. 모든 Task → sprint-backlog 이관
2. Milestone 삭제
3. Sprint Issue 종료 (취소 사유 기록)
