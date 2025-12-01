---
name: audit-issues
description: 이슈관리 보드의 전체 이슈 품질 감사. Use when (1) 중복 이슈 검토, (2) 필수 필드 누락 검토, (3) Projects 미연결 검토, (4) task 이슈 작업량 미할당 검토.
tools: [Bash, Read, Write]
---

> **🔔 시스템 메시지**: 이 Skill이 호출되면 `[SAX] Skill: audit-issues 호출` 시스템 메시지를 첫 줄에 출력하세요.

# audit-issues Skill

> 이슈관리 보드 전체 이슈 품질 감사 Skill

## Purpose

`이슈관리` Projects 보드의 모든 이슈를 검토하여 품질 문제를 발견합니다.

| 검토 항목 | 설명 |
|----------|------|
| **중복 이슈** | 제목/내용이 유사한 이슈 탐지 |
| **필수 필드 누락** | 타입, 우선순위 등 필수 속성 미설정 |
| **Projects 미연결** | 이슈관리 보드에 연결되지 않은 이슈 |
| **작업량 미할당** | task 타입인데 작업량이 설정되지 않은 이슈 |

## Quick Start

```bash
# 1. 이슈관리 프로젝트 전체 이슈 조회
gh api graphql -f query='...'

# 2. 4가지 감사 항목 체크

# 3. 결과 리포트 출력
```

## 감사 항목 상세

### 1. 중복 이슈 검토

**탐지 기준**:
- 제목 유사도 80% 이상
- 본문 키워드 70% 이상 일치
- 동일 레포지토리 내 Open 상태 이슈 간 비교

**출력 예시**:
```markdown
⚠️ 중복 의심 이슈 발견:
- #12 "로그인 버그 수정" ↔ #45 "로그인 오류 해결" (유사도: 85%)
```

### 2. 필수 필드 누락 검토

**필수 필드** (이슈관리 보드 기준):

| 필드 | 필수 여부 | 설명 |
|------|----------|------|
| **타입** | ✅ 필수 | 에픽, 버그, 태스크 중 선택 |
| **우선순위** | ✅ 필수 | P0~P4 중 선택 |
| **Status** | ✅ 필수 | 워크플로우 상태 |
| **기술영역** | 권장 | 담당 영역 (복수 선택 가능) |

**출력 예시**:
```markdown
❌ 필수 필드 누락:
- #23 "API 개선" - 타입 미설정
- #34 "버그 수정" - 우선순위 미설정
```

### 3. Projects 미연결 검토

**탐지 대상**:
- `semicolon-devteam` 조직의 모든 레포지토리
- Open 상태 이슈 중 `이슈관리` 프로젝트에 연결되지 않은 항목

**출력 예시**:
```markdown
⚠️ Projects 미연결 이슈:
- semicolon-app#56 "새 기능 요청"
- semicolon-web#12 "UI 개선"
```

### 4. task 이슈 작업량 미할당 검토

**탐지 기준**:
- 타입 = "태스크"
- 작업량 필드 = null 또는 미설정

**출력 예시**:
```markdown
❌ 작업량 미할당 태스크:
- #45 "컴포넌트 리팩토링" - 작업량 미설정
- #67 "테스트 코드 추가" - 작업량 미설정
```

## GraphQL 쿼리

### 프로젝트 전체 이슈 조회

```graphql
query {
  organization(login: "semicolon-devteam") {
    projectV2(number: 1) {
      items(first: 100) {
        nodes {
          id
          content {
            ... on Issue {
              number
              title
              body
              state
              repository {
                name
              }
            }
          }
          fieldValues(first: 10) {
            nodes {
              ... on ProjectV2ItemFieldSingleSelectValue {
                name
                field { ... on ProjectV2SingleSelectField { name } }
              }
              ... on ProjectV2ItemFieldNumberValue {
                number
                field { ... on ProjectV2Field { name } }
              }
            }
          }
        }
      }
    }
  }
}
```

### 레포지토리별 Open 이슈 조회

```bash
gh issue list --repo semicolon-devteam/{repo} --state open --json number,title,body,projectItems
```

## Output Format

### 정상 결과

```markdown
[SAX] Skill: audit-issues 완료

## 📊 이슈 감사 결과

**검토 대상**: 이슈관리 보드 (42개 이슈)
**검토 일시**: 2025-12-01

### ✅ 감사 통과
- 중복 이슈: 없음
- 필수 필드 누락: 없음
- Projects 미연결: 없음
- 작업량 미할당 태스크: 없음

**총평**: 모든 이슈가 품질 기준을 충족합니다.
```

### 문제 발견 시

```markdown
[SAX] Skill: audit-issues 완료

## 📊 이슈 감사 결과

**검토 대상**: 이슈관리 보드 (42개 이슈)
**검토 일시**: 2025-12-01

### ⚠️ 발견된 문제

#### 1. 중복 의심 이슈 (2건)
| 이슈 A | 이슈 B | 유사도 |
|--------|--------|--------|
| #12 로그인 버그 | #45 로그인 오류 | 85% |

#### 2. 필수 필드 누락 (3건)
| 이슈 | 누락 필드 |
|------|----------|
| #23 API 개선 | 타입 |
| #34 버그 수정 | 우선순위 |

#### 3. Projects 미연결 (1건)
| 레포지토리 | 이슈 |
|-----------|------|
| semicolon-app | #56 새 기능 요청 |

#### 4. 작업량 미할당 태스크 (2건)
| 이슈 | 제목 |
|------|------|
| #45 | 컴포넌트 리팩토링 |
| #67 | 테스트 코드 추가 |

### 📋 권장 조치
1. 중복 이슈 검토 후 병합 또는 닫기
2. 필수 필드 설정 완료
3. 미연결 이슈를 이슈관리 보드에 추가
4. 태스크 이슈에 작업량 할당
```

## SAX Message

```markdown
[SAX] Skill: audit-issues 호출

[SAX] Audit: 이슈관리 보드 조회 중... (42개 이슈)

[SAX] Audit: 4가지 감사 항목 검토 중...
- 중복 이슈 검토 ✅
- 필수 필드 검토 ✅
- Projects 연결 검토 ✅
- 작업량 할당 검토 ✅

[SAX] Skill: audit-issues 완료 (문제 {n}건 발견)
```

## Related

- [detect-blockers Skill](../detect-blockers/SKILL.md) - 블로커 탐지
- [sync-project-status Skill](../sync-project-status/SKILL.md) - 프로젝트 상태 동기화
- [project-board Skill (sax-next)](../../sax-next/skills/project-board/SKILL.md) - Projects 보드 관리

## References

For detailed documentation, see:

- [Audit Checks](references/audit-checks.md) - 4가지 감사 항목 상세 로직
- [GraphQL Queries](references/graphql-queries.md) - API 쿼리 전체 목록
