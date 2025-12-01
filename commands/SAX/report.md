# /SAX:report - 리포트 생성 커맨드

> 다양한 유형의 리포트를 생성하고 공유

## 사용법

```bash
/SAX:report <type> [options]
```

## Report Types

### weekly - 주간 리포트

```bash
/SAX:report weekly
/SAX:report weekly --slack
/SAX:report weekly --slack #_협업
```

**내용**:
- 이번 주 완료 Task
- 다음 주 예정 Task
- 주요 이슈/블로커
- Sprint 진행률

---

### member - 인원별 리포트

```bash
/SAX:report member --all
/SAX:report member @kyago
/SAX:report member @kyago @Garden
```

**내용**:
- 담당자별 할당/완료 현황
- 완료율 및 용량 대비
- 진행중/대기 Task 목록

---

### blocker - 블로커 리포트

```bash
/SAX:report blocker
/SAX:report blocker --notify
```

**내용**:
- Critical/Warning 블로커 목록
- 지연 일수 및 원인
- 담당자 정보

---

### velocity - Velocity 리포트

```bash
/SAX:report velocity
/SAX:report velocity --sprints 5
```

**내용**:
- 최근 Sprint별 Velocity
- 평균 Velocity
- 트렌드 분석

---

## Options

### --slack

리포트를 Slack으로 전송합니다.

```bash
/SAX:report weekly --slack
/SAX:report weekly --slack #_협업
```

### --sprint

특정 Sprint 대상으로 리포트를 생성합니다.

```bash
/SAX:report weekly --sprint "Sprint 23"
```

### --format

출력 형식을 지정합니다.

```bash
/SAX:report weekly --format markdown
/SAX:report weekly --format json
```

---

## 출력 예시

### 주간 리포트

```markdown
# 📅 주간 리포트 (Week 49)

**기간**: 2024-12-02 ~ 2024-12-08
**Sprint**: Sprint 23

## ✅ 이번 주 완료
- #450 로그인 리팩토링 (@kyago)
- #451 에러 핸들링 (@Garden)
- #452 테스트 코드 (@Roki)

## 🔄 진행중
- #456 댓글 API (@kyago) - 70%
- #457 댓글 UI (@Garden) - 50%

## ⏳ 다음 주 예정
- #458 알림 연동
- #459 푸시 설정

## ⚠️ 주요 이슈
- #234 의존성 미해결 (3일 지연)

## 📊 Sprint 진행률
████████░░ 78%
```

### 인원별 리포트

```markdown
# 👤 @kyago 업무 현황

**Sprint**: Sprint 23
**할당**: 12pt | **완료**: 8pt | **완료율**: 67%

## ✅ 완료 (3)
- [x] #450 로그인 리팩토링 (3pt)
- [x] #451 에러 핸들링 (2pt)
- [x] #452 테스트 코드 (3pt)

## 🔄 진행중 (1)
- [ ] #456 댓글 API (3pt) - 70%

## ⏳ 대기 (1)
- [ ] #458 알림 연동 (1pt)
```

## Routing

이 커맨드는 `progress-tracker` Agent에게 위임됩니다.

```markdown
[SAX] Orchestrator: 의도 분석 완료 → 리포트 생성

[SAX] Agent 위임: progress-tracker (사유: {type} 리포트 생성)
```

## 연관 Skills

- `generate-progress-report`: Sprint 진행도 리포트
- `generate-member-report`: 인원별 리포트
- `detect-blockers`: 블로커 리포트
- `calculate-velocity`: Velocity 리포트
