# /SAX:sprint - Sprint 관리 커맨드

> Sprint 생성, 할당, 종료를 위한 통합 커맨드

## 사용법

```bash
/SAX:sprint <action> [options]
```

## Actions

### create - Sprint 생성

```bash
/SAX:sprint create "Sprint 23" --start 2024-12-02 --end 2024-12-13
/SAX:sprint create "Sprint 23" --goals "댓글 기능 완성, 알림 연동"
```

**파라미터**:
- `name`: Sprint 이름 (필수)
- `--start`: 시작일 (YYYY-MM-DD)
- `--end`: 종료일 (YYYY-MM-DD)
- `--goals`: Sprint 목표 (쉼표 구분)

**동작**:
1. GitHub Milestone 생성
2. Sprint Issue 생성
3. sprint-current 라벨 설정

---

### add - Task 할당

```bash
/SAX:sprint add #123 #124 #125 --to "Sprint 23"
/SAX:sprint add #123 --to current
```

**파라미터**:
- `task_numbers`: 할당할 Task 번호들
- `--to`: 대상 Sprint (이름 또는 "current")

**동작**:
1. Task에 sprint-N 라벨 추가
2. Milestone 연결
3. 용량 체크 및 경고

---

### status - Sprint 현황

```bash
/SAX:sprint status
/SAX:sprint status "Sprint 23"
```

**파라미터**:
- `name`: Sprint 이름 (선택, 기본: 현재 Sprint)

**동작**:
1. Sprint 진행도 조회
2. Task 상태별 집계
3. 리포트 출력

---

### close - Sprint 종료

```bash
/SAX:sprint close "Sprint 23"
/SAX:sprint close "Sprint 23" --carry-to "Sprint 24"
```

**파라미터**:
- `name`: Sprint 이름 (필수)
- `--carry-to`: 미완료 Task 이관 대상 Sprint

**동작**:
1. 완료/미완료 집계
2. Velocity 계산
3. 회고 생성
4. Milestone 종료
5. 미완료 Task 이관

---

## 예시

### Sprint 전체 워크플로우

```bash
# 1. Sprint 생성
/SAX:sprint create "Sprint 23" --start 2024-12-02 --end 2024-12-13

# 2. Task 할당
/SAX:sprint add #123 #124 #125 --to "Sprint 23"

# 3. 진행중 현황 확인
/SAX:sprint status

# 4. Sprint 종료
/SAX:sprint close "Sprint 23" --carry-to "Sprint 24"
```

## Routing

이 커맨드는 `sprint-master` Agent에게 위임됩니다.

```markdown
[SAX] Orchestrator: 의도 분석 완료 → Sprint 관리

[SAX] Agent 위임: sprint-master (사유: Sprint {action} 요청)
```
