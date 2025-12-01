---
name: sax-help
description: |
  SAX-PM 도움말 표시. Use when (1) 도움말 요청,
  (2) 사용법 문의, (3) 기능 목록 조회.
tools: [Read]
model: inherit
---

> **시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: sax-help 호출` 메시지를 첫 줄에 출력하세요.

# sax-help Skill

> SAX-PM 패키지 도움말

## Purpose

SAX-PM 패키지의 기능과 사용법을 안내합니다.

## Output

```markdown
# 📚 SAX-PM 도움말

> PM/프로젝트 매니저를 위한 SAX 패키지

## 🎯 주요 기능

### Sprint 관리
- **Sprint 생성**: 2주 단위 Sprint 계획
- **Task 할당**: Backlog → Sprint 할당
- **Sprint 종료**: 회고 및 Velocity 계산

### 진행도 추적
- **진행 현황**: Sprint 완료율 실시간 확인
- **인원별 리포트**: 담당자별 업무 현황
- **블로커 감지**: 지연/차단 Task 자동 감지

### Roadmap
- **Roadmap 생성**: Epic 기반 타임라인
- **Mermaid 차트**: Gantt 차트 시각화

## 📋 Commands

### /SAX:sprint
```bash
# Sprint 생성
/SAX:sprint create "Sprint 23" --start 2024-12-02 --end 2024-12-13

# Task 할당
/SAX:sprint add #123 #124 --to "Sprint 23"

# Sprint 현황
/SAX:sprint status

# Sprint 종료
/SAX:sprint close "Sprint 23"
```

### /SAX:progress
```bash
# 현재 Sprint 진행도
/SAX:progress

# 특정 Sprint
/SAX:progress --sprint "Sprint 23"
```

### /SAX:report
```bash
# 주간 리포트
/SAX:report weekly

# 인원별 리포트
/SAX:report member --all
/SAX:report member @kyago

# Slack 전송
/SAX:report weekly --slack
```

### /SAX:roadmap
```bash
# Roadmap 생성
/SAX:roadmap generate

# Mermaid 형식
/SAX:roadmap --format mermaid
```

## 🤖 Agents

| Agent | 역할 |
|-------|------|
| orchestrator | 요청 라우팅 |
| sprint-master | Sprint 관리 |
| progress-tracker | 진행도 추적 |
| roadmap-planner | Roadmap 관리 |

## ⚡ Skills

| Skill | 기능 |
|-------|------|
| create-sprint | Sprint 생성 |
| close-sprint | Sprint 종료 |
| assign-to-sprint | Task 할당 |
| calculate-velocity | Velocity 계산 |
| generate-progress-report | 진행도 리포트 |
| generate-member-report | 인원별 리포트 |
| detect-blockers | 블로커 감지 |
| generate-roadmap | Roadmap 생성 |
| sync-project-status | Projects 동기화 |

## 💡 예시

### "Sprint 23 현황 알려줘"
→ progress-tracker가 Sprint 진행도 리포트 생성

### "kyago 업무 현황"
→ progress-tracker가 인원별 리포트 생성

### "다음 분기 로드맵 만들어줘"
→ roadmap-planner가 Mermaid Gantt 차트 생성

## 🔗 관련 패키지

- **SAX-PO**: Epic/Task 생성 (기획)
- **SAX-Core**: 공통 Skill (notify-slack 등)

---
*SAX-PM v0.1.0*
```

## 완료 메시지

```markdown
[SAX] Skill: sax-help 완료

SAX-PM 도움말을 표시했습니다.
자세한 사용법은 `/SAX:help` 커맨드를 사용하세요.
```
