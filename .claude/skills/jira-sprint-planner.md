---
name: jira-sprint-planner
description: SwiftSight 스프린트 플래닝 — Jira 카드 CRUD, 일정 관리, HTML 캘린더 생성
---

# SwiftSight Sprint Planner Skill

Jira에서 스프린트 플래닝을 수행하는 통합 스킬. Story 카드(SS), Backend 개발카드(ABE), Frontend 개발카드(AFE), QA 카드(AP) 를 CRUD하고, HTML Gantt 캘린더를 생성/관리한다.

## 프로젝트 구조

| Board | Project Key | 용도 | 담당 |
|-------|-------------|------|------|
| SwiftSight (SS) | `SS` | Story/Task 카드 (기능 단위) | PM (황태현) |
| AIRS Backend (ABE) | `ABE` | BE 개발카드 | 박태웅, 승현수 |
| AIRS Frontend (AFE) | `AFE` | FE 개발카드 | 박이호 |
| AIRS PQ (AP) | `AP` | QA 피쳐 테스트 카드 | 전영선 (Milla) |

## Jira Cloud 정보

- **Cloud ID / Site URL**: `airsmed.atlassian.net`
- **Epic**: SS-172 `[Sprint 2026/03/19 ~ 2026/05/08 - v 2.0.0.0]`
- **Epic Link Field**: `customfield_10014`
- **Technical Feasibility Score Field**: `customfield_10175` (ABE 보드에서 스토리 포인트 대신 사용)

### SS 프로젝트 Description 필드 (이슈 타입별 다름!)

| 이슈 타입 | 필드 이름 | Custom Field ID | 형식 |
|-----------|----------|----------------|------|
| **Story** | SwiftSight Story Description | `customfield_10669` | ADF |
| **Task** | SwiftSight Task Description | `customfield_10701` | ADF |

**주의:**
- `description` 표준 필드에 써도 API에서는 저장되지만 **Jira UI에 표시되지 않음**
- 반드시 이슈 타입에 맞는 커스텀 필드에 ADF 형식으로 작성해야 함
- markdown `contentFormat`은 이 필드에서 거부됨 → ADF JSON 직접 구성 필요
- ABE/AFE 보드 Task 카드는 표준 `description` 필드 사용 (markdown 가능)

## 팀 Account IDs

| 이름 | Account ID | 역할 |
|------|-----------|------|
| 황태현 | `6094ce17000224006ac39b0a` | PM |
| 박이호 | `5f55c49e10d187006f16e6f7` | Frontend |
| 박태웅 | `62d4d4ff04a004286c01b362` | Backend |
| 승현수 | `712020:f5d706b3-8c36-4369-8555-277754eb8ae9` | Backend |
| 전영선 (Milla) | `712020:0a1ee268-21ad-492b-8149-2240b89221dc` | QA |

## 카드 CRUD 패턴

### 1. Story 카드 조회 (SS)

```
# Epic 하위 전체 Story 카드 조회
JQL: "Epic Link" = SS-172 ORDER BY key ASC
Fields: summary, status, issuetype, assignee, duedate, issuelinks
```

### 2. 개발카드 조회 (ABE/AFE)

```
# 특정 개발자의 미완료 카드 조회
JQL: project = ABE AND assignee = "{accountId}" AND status != Done AND status != Obsoleted ORDER BY key ASC
Fields: summary, status, customfield_10175, duedate, issuelinks

# Technical Feasibility Score로 스토리 포인트 확인
Field: customfield_10175 (Number)
```

### 3. 개발카드 생성

```javascript
// BE 개발카드 생성
createJiraIssue({
  cloudId: "airsmed.atlassian.net",
  projectKey: "ABE",  // or "AFE"
  issueTypeName: "Task",
  summary: "[SS-XXX] 작업 제목",
  description: "## 설명\n...",
  assignee_account_id: "담당자_account_id",
  contentFormat: "markdown",
  additional_fields: {
    duedate: "2026-04-XX",
    labels: ["SS-XXX", "카테고리"],
    customfield_10175: 8  // Technical Feasibility Score
  }
})
```

### 4. SS Story/Task Description 작성

SS 프로젝트 카드의 Description은 **표준 description이 아닌 커스텀 필드**에 **ADF 형식**으로 작성해야 UI에 표시된다.

```javascript
// SS Story 카드 Description 작성
editJiraIssue({
  cloudId: "airsmed.atlassian.net",
  issueIdOrKey: "SS-XXX",
  fields: {
    "customfield_10669": {  // Story → customfield_10669
      "version": 1, "type": "doc",
      "content": [
        {"type": "heading", "attrs": {"level": 1}, "content": [{"type": "text", "text": "제목"}]},
        {"type": "paragraph", "content": [{"type": "text", "text": "내용"}]},
        {"type": "bulletList", "content": [
          {"type": "listItem", "content": [{"type": "paragraph", "content": [{"type": "text", "text": "항목"}]}]}
        ]}
      ]
    }
  }
})

// SS Task 카드 Description 작성
editJiraIssue({
  cloudId: "airsmed.atlassian.net",
  issueIdOrKey: "SS-XXX",
  fields: {
    "customfield_10701": { ... }  // Task → customfield_10701
  }
})
```

**주의:** `contentFormat: "markdown"` 은 이 필드에서 거부됨. ADF JSON 직접 작성 필수.

### 5. QA 카드 생성 (AP 보드)

```javascript
// QA 피쳐 테스트 카드 — AP 프로젝트에 "QA" 이슈 타입으로 생성
createJiraIssue({
  cloudId: "airsmed.atlassian.net",
  projectKey: "AP",
  issueTypeName: "QA",  // AP 프로젝트 전용 이슈 타입
  summary: "[SS-XXX] 피쳐 테스트 (N일)",
  assignee_account_id: "712020:0a1ee268-21ad-492b-8149-2240b89221dc",  // 전영선
  additional_fields: {
    duedate: "2026-04-XX",
    priority: { name: "Highest" },
    labels: ["feature-test", "SS-XXX"]
  }
})
```

### 5. 카드 간 링크

```javascript
// Story ↔ Dev 카드 링크
createIssueLink({
  cloudId: "airsmed.atlassian.net",
  inwardIssue: "SS-XXX",    // Story
  outwardIssue: "ABE-XX",   // Dev 카드
  type: "Relates"            // or "Blocks"
})
```

### 6. Due Date 업데이트

```javascript
editJiraIssue({
  cloudId: "airsmed.atlassian.net",
  issueIdOrKey: "ABE-XX",
  fields: { duedate: "2026-04-XX" }
})
```

### 7. 상태 전환

```javascript
// 먼저 가능한 transition 조회
getTransitionsForJiraIssue({ issueIdOrKey: "SS-XXX" })

// 주요 Transition IDs (SS 프로젝트)
// 8: Obsoleted, 9: Backlog, 20: Planning, 21: Ready for Dev
// 22: In Development, 23: Ready for Test, 24: In Test, 27: Done

transitionJiraIssue({
  issueIdOrKey: "SS-XXX",
  transition: { id: "8" }  // Obsoleted
})
```

## Cross-Project 제약

- **parent 설정**: 같은 프로젝트 내에서만 가능. ABE/AFE 카드를 SS Story의 child로 설정 불가
- **Epic Link**: `customfield_10014`로 SS-172 Epic에 연결 가능 (같은 프로젝트 내)
- **Plans Timeline**: SS 프로젝트만 기본 표시. ABE/AFE를 보려면 Plans Settings > Issue Sources에 추가 필요

## HTML Gantt 캘린더

파일: `/Users/taehyun/github/HwangTaehyun/jira-study/sprint-planning.html`

### 구조

```
1. Story Cards (SS) — 최상단, 전체 Story 타임라인
2. 박이호 (Frontend) — AFE-* 개발카드
3. 박태웅 (Backend) — ABE-* 개발카드
4. 승현수 (Backend) — ABE-* 개발카드
5. 전영선 (QA) — AP-* QA 카드
```

### 규칙

- 개발자 행에 SS-* 카드번호 사용 금지 → 반드시 ABE-*/AFE-*/AP-* 개발카드 번호 사용
- 같은 개발자 행에서 기간 겹침 금지
- BE 선 개발 → FE 개발 → 1일 Buffer → QA 테스트 순서
- 버퍼: 전체 19일의 20% = 4일, 마지막에 몰빵하지 말고 중간에 배치
- 바 클릭 시 Jira 카드로 이동 (`data-jira` 속성)
- Column Width 슬라이더로 컬럼 너비 조정 가능

### Working Days Index

```
0=3/27(금), 1=3/30(월), 2=3/31(화), 3=4/1(수), 4=4/2(목), 5=4/3(금),
6=4/6(월), 7=4/7(화), 8=4/8(수), 9=4/9(목), 10=4/10(금),
11=4/13(월), 12=4/14(화), 13=4/15(수), 14=4/16(목), 15=4/17(금),
16=4/20(월), 17=4/21(화), 18=4/22(수)
```

### Task 데이터 형식

```javascript
// [label, colorVar, startDayIdx, endDayIdx, points, tooltip, barClass?, jiraKey?]
['ABE-68 SS-419 (4p)', '--c419', 1, 1, 4, '설명...', null, 'ABE-68']
```

## Feasibility 검증

개발자별 capacity 계산:
```
유효 일수 = 19일 - 휴가일수
작업 가능 일수 = 유효 일수 * 0.8 (20% 버퍼)
작업 가능 포인트 = 작업 가능 일수 * 8 (1일 = 8 story points)
활용률 = 총 SP / 작업 가능 포인트
```

**Technical Feasibility Score** (`customfield_10175`)를 Jira에서 직접 조회하여 검증할 것. HTML에 추정치를 넣지 말고 Jira 실데이터 사용.
