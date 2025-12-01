# /SAX:sprint - Sprint 관리 커맨드

> Iteration 기반 Sprint 관리를 위한 통합 커맨드

## 사용법

```bash
/SAX:sprint <action> [options]
```

## Iteration 구조

```yaml
주기: 1주 (7일)
시작: 월요일
명명: "{월} {주차}/{월 총주차}"  # 예: "12월 1/4"
```

> GitHub Projects의 Iteration은 자동 생성됩니다. Sprint는 해당 Iteration을 "활성화"하고 목표를 설정하는 개념입니다.

---

## Actions

### create - Sprint 활성화

```bash
/SAX:sprint create "12월 1/4" --goals "댓글 기능 완성, 알림 연동"
/SAX:sprint create current --goals "로그인 개선"
```

**파라미터**:

- `iteration_title`: Iteration 이름 (필수, 예: "12월 1/4")
- `--goals`: Sprint 목표 (쉼표 구분)

**동작**:

1. Iteration 존재 확인 (GraphQL)
2. Sprint Issue 생성 (docs 레포)
3. sprint-current 라벨 설정

---

### add - Task 할당

```bash
/SAX:sprint add #123 #124 #125 --to "12월 1/4"
/SAX:sprint add #123 --to current
```

**파라미터**:

- `task_numbers`: 할당할 Task 번호들
- `--to`: 대상 Iteration 이름 또는 "current"

**동작**:

1. Task의 Iteration 필드 설정 (GraphQL)
2. 용량 체크 및 경고
3. 과할당 시 알림

---

### status - Sprint 현황

```bash
/SAX:sprint status
/SAX:sprint status "12월 1/4"
```

**파라미터**:

- `iteration_title`: Iteration 이름 (선택, 기본: 현재 Iteration)

**동작**:

1. Iteration Task 현황 조회
2. 상태별 집계
3. 리포트 출력

---

### close - Sprint 종료

```bash
/SAX:sprint close "12월 1/4"
/SAX:sprint close "12월 1/4" --carry-to "12월 2/4"
```

**파라미터**:

- `iteration_title`: Iteration 이름 (필수)
- `--carry-to`: 미완료 Task 이관 대상 Iteration

**동작**:

1. 완료/미완료 집계
2. Velocity 계산 (작업량 필드 기준)
3. 회고 생성
4. Sprint Issue 종료
5. 미완료 Task Iteration 이관

---

## 예시

### Sprint 전체 워크플로우

```bash
# 1. Sprint 활성화 (Iteration은 이미 존재)
/SAX:sprint create "12월 1/4" --goals "댓글 기능 완성"

# 2. Task 할당 (Iteration 필드 설정)
/SAX:sprint add #123 #124 #125 --to "12월 1/4"

# 3. 진행중 현황 확인
/SAX:sprint status

# 4. Sprint 종료 (미완료 이관)
/SAX:sprint close "12월 1/4" --carry-to "12월 2/4"
```

### Iteration 목록 확인

```bash
# 활성 Iteration 조회
/SAX:sprint list
```

---

## Routing

이 커맨드는 `sprint-master` Agent에게 위임됩니다.

```markdown
[SAX] Orchestrator: 의도 분석 완료 → Sprint 관리

[SAX] Agent 위임: sprint-master (사유: Sprint {action} 요청)
```

## 연관 Skills

- `create-sprint`: Sprint 활성화
- `assign-to-sprint`: Task Iteration 할당
- `close-sprint`: Sprint 종료 및 회고
