---
name: onboarding
description: 신규 PM 온보딩 프로세스 시작
---

# /SAX:onboarding Command

신규 PM(Project Manager)을 위한 온보딩 프로세스를 시작합니다.

## Trigger

- `/SAX:onboarding` 명령어
- "처음이에요", "신규", "온보딩 시작" 키워드

## Action

`onboarding-master` Agent를 호출하여 5단계 온보딩 프로세스를 안내합니다.

## Process

```text
1. 환경 진단 (skill:health-check)
2. 조직 참여 확인 (Slack, GitHub, GitHub Projects)
3. SAX 개념 학습 (PM 워크플로우)
4. 실습 (Sprint 생성 → 진행도 추적 → 보고서 생성)
5. 온보딩 완료 및 메타데이터 저장
```

## Expected Output

```markdown
[SAX] Orchestrator: 의도 분석 완료 → 온보딩 요청

[SAX] Agent: onboarding-master 호출 (트리거: /SAX:onboarding)

=== SAX-PM 온보딩 프로세스 시작 ===

Phase 0: 환경 진단
...

Phase 2: PM 워크플로우
1. Sprint 생성 ("/SAX:sprint create")
2. Epic/Task 할당 ("skill:assign-task")
3. 진행도 추적 ("/SAX:progress")
4. 보고서 생성 ("/SAX:report")
5. Sprint 종료 ("skill:close-sprint")
```

## Related

- [onboarding-master Agent](../../agents/onboarding-master/onboarding-master.md)
- [health-check Skill](../../skills/health-check/SKILL.md)
