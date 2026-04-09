# Jira Admin Guide — 커스텀 필드 & 프로젝트 설정

---

## 1. Custom Field Configuration

### 접근 경로

```
Jira Settings → Issues → Custom Fields → [필드 선택] → Configure
URL: https://airsmed.atlassian.net/secure/admin/ConfigureCustomField!default.jspa?customFieldId={ID}
```

예: `customFieldId=10669` = SwiftSight Story Description

### Custom Field Configuration이란?

하나의 커스텀 필드가 **프로젝트/이슈 타입에 따라 다르게 동작**하도록 설정하는 기능.

### 핵심 개념: Context

**Context** = "이 필드가 어디서 어떻게 동작할지" 정의하는 범위 설정

```
SwiftSight Story Description (customfield_10669)
├── Context 1: "SS 프로젝트 Story"
│   ├── 적용 범위: SS 프로젝트, Story 이슈 타입
│   └── Default Value: (스토리 템플릿 ADF)
├── Context 2: "AT 프로젝트 Story"
│   ├── 적용 범위: AT 프로젝트, Story 이슈 타입
│   └── Default Value: (다른 템플릿 또는 동일)
└── Global Context (기본)
    ├── 적용 범위: 나머지 전체
    └── Default Value: (없음)
```

### Context 설정 항목

| 항목 | 설명 | 예시 |
|------|------|------|
| **Name** | Context 식별 이름 | "SwiftSight Story Context" |
| **Applicable issue types** | 이 Context가 적용될 이슈 타입 | Story만, 또는 All |
| **Applicable projects** | 이 Context가 적용될 프로젝트 | SS만, 또는 Global |
| **Default Value** | 이슈 생성 시 필드에 자동 채워지는 값 | ADF 템플릿 (User Needs, Solutions...) |

### Default Value

이슈를 **새로 생성할 때** 필드에 미리 채워지는 값.

SS의 Story Description 필드에 Default Value가 설정되어 있으면:
→ SS에서 Story를 만들 때 자동으로 "User Needs / Solutions / Use Cases..." 템플릿이 채워짐
→ 작성자가 빈 칸부터 시작하지 않고, 구조에 맞게 내용을 채우면 됨

### 동일 필드, 다른 프로젝트에서 다른 동작

```
customfield_10669 (SwiftSight Story Description)

SS 프로젝트에서 Story 생성:
→ Context "SS Story" 매칭
→ Default Value: User Needs → Solutions → FR → 참고 템플릿 자동 로드

AT 프로젝트에서 Story 생성:
→ Context 없음 (아직 설정 안 함)
→ Default Value: 비어있음
→ 또는 Global Context의 Default Value 적용
```

### AT에 적용하려면

1. `ConfigureCustomField` 페이지에서 **새 Context 추가**
2. Applicable projects: **AT** 선택
3. Applicable issue types: **Story** 선택
4. Default Value: SS와 동일한 템플릿 설정 (또는 AT에 맞게 수정)

이렇게 하면 AT에서 Story를 만들 때도 SS와 동일한 스펙 템플릿이 자동으로 채워진다.

---

## 2. 프로젝트 설정 구조 (Work Items)

### 전체 계층

```
프로젝트 (Project)
├── Work Items
│   ├── Types          — 사용할 이슈 타입 (Story, Task, Bug...)
│   ├── Layout         — 카드 상세 화면 필드 배치 순서
│   ├── Screens        — 생성/편집/전환 시 보여줄 필드 세트
│   ├── Fields         — 필드별 속성 (필수/선택, 렌더러)
│   ├── Collectors     — 외부 소스에서 이슈 자동 수집
│   └── Security       — 이슈 접근 제한
├── Workflows          — 이슈 상태 전환 규칙
├── Versions           — 릴리즈 버전 관리
├── Components         — 기능 모듈 분류
└── Apps               — 연동 앱
```

### Screen 계층

```
Field (전역)
  → Screen (필드 목록)
    → Screen Scheme (생성/편집/전환별 Screen 매핑)
      → Issue Type Screen Scheme (이슈 타입별 Screen Scheme 매핑)
        → Project (프로젝트에 적용)
```

### Field Configuration 계층

```
Field (전역)
  → Field Configuration (필드별 필수/숨김/렌더러 설정)
    → Field Configuration Scheme (이슈 타입별 Field Configuration 매핑)
      → Project (프로젝트에 적용)
```

### Workflow 계층

```
Workflow (상태 전환 규칙)
  → Workflow Scheme (이슈 타입별 Workflow 매핑)
    → Project (프로젝트에 적용)
```

---

## 3. 커스텀 필드 핵심 ID 참조 (airsmed.atlassian.net)

### SS 프로젝트 전용 필드

| 필드 이름 | Field ID | 이슈 타입 | 용도 |
|-----------|----------|----------|------|
| SwiftSight Story Description | `customfield_10669` | Story | 스토리 스펙 템플릿 |
| SwiftSight Task Description | `customfield_10701` | Task | 태스크 스펙 |
| 업무 요약 | `customfield_10118` | Task | 배경/내용/주체 |
| Task Description | `customfield_10162` | Task | 상세 작업 내용 |
| 설계/구현/QA | `customfield_10142` | Task | 개발 프로세스 |

### ABE 보드 필드

| 필드 이름 | Field ID | 용도 |
|-----------|----------|------|
| Technical Feasibility Score | `customfield_10175` | 스토리 포인트 대신 사용 |

### 공통 필드

| 필드 이름 | Field ID | 용도 |
|-----------|----------|------|
| Epic Link | `customfield_10014` | Epic 연결 |
| Start date | `customfield_10015` | 시작일 |
| Story Points | `customfield_10016` | 표준 SP (사용 안 함) |
| Sprint | `customfield_10020` | 스프린트 연결 |

---

## 4. 커스텀 필드를 다른 프로젝트에 적용하는 전체 과정

### 상황: SS의 "SwiftSight Story Description"을 AT에도 적용

```
Step 1: Context 추가 (Default Value 설정)
  ConfigureCustomField → Add Context → AT + Story → Default Value 설정
  
Step 2: Screen에 필드 추가
  AT Project Settings → Screens → Story Screen → Add Field → "SwiftSight Story Description"
  
Step 3: Layout에서 위치 조정
  AT Project Settings → Layout → Story → 필드 순서 조정
  
Step 4: (선택) Field Configuration에서 필수/선택 설정
  AT Project Settings → Fields → "SwiftSight Story Description" → Required/Optional
```

### 확인 방법

AT에서 Story 카드를 새로 만들어 보기:
- "SwiftSight Story Description" 필드가 보이는지?
- Default Value(템플릿)가 자동 로드되는지?
- 저장 후 카드 상세에서 내용이 표시되는지?

---

## 5. Screen Scheme 만들기

### 개념

Screen Scheme = **"언제 어떤 Screen을 쓸지"** 매핑.

이슈를 생성(Create)/편집(Edit)/조회(View)할 때 각각 다른 Screen을 보여줄 수 있다.

### 생성 화면

```
경로: Jira Admin → Issues → Screen Schemes → Add Screen Scheme
```

| 필드 | 설명 | 예시 |
|------|------|------|
| **Name** | Screen Scheme 이름 | "AI Transform: Story Scheme" |
| **Description** | 설명 (선택) | |
| **Default Screen** | 별도 매핑이 없는 작업에 사용할 Screen | "AI Transformation: Story Screen" |

**Default Screen**이란: Create/Edit/View 중 별도 매핑을 안 한 작업에 대해 사용할 Screen.

```
AI Transform: Story Scheme
├── Create Issue → (매핑 없음) → Default Screen 사용
├── Edit Issue   → (매핑 없음) → Default Screen 사용
└── View Issue   → (매핑 없음) → Default Screen 사용

Default Screen = "AI Transformation: Story Screen"
→ 결과: 생성/편집/조회 모두 같은 Screen 사용
```

나중에 Create 시에만 간소화된 Screen을 쓰고 싶으면 별도 매핑을 추가하면 된다.

---

## 6. Work Type Screen Scheme (Issue Type Screen Scheme)

### 개념

**최상위 매핑** — 이슈 타입별로 어떤 Screen Scheme을 쓸지 결정하는 곳.

```
경로: Jira Admin → Issues → Work type screen schemes
      (= Issue Type Screen Schemes)
```

### 구조

```
Work Type Screen Scheme (프로젝트에 1개 적용)
├── Story   → Story Screen Scheme → Story Screen → 필드 목록
├── Task    → Task Screen Scheme  → Task Screen  → 필드 목록
├── Bug     → Bug Screen Scheme   → Bug Screen   → 필드 목록
└── Default → Default Screen Scheme (매핑 안 된 이슈 타입용)
```

### AT 적용 예시

**현재 상태:**

```
AT: Kanban Issue Type Screen Scheme (1 PROJECT)
└── Default → AI Transform: Story Scheme
    → 모든 이슈 타입이 Story Screen을 공유
```

**목표 상태 (Story/Task 분리):**

```
AT: Kanban Issue Type Screen Scheme
├── Story   → AI Transform: Story Scheme (Story Screen)
├── Task    → AI Transform: Task Scheme (Task Screen)  ← 추가
└── Default → AI Transform: Story Scheme (나머지)
```

### Task 분리 절차

1. **Task Screen 생성** — "AI Transformation: Task Screen" 
   - SwiftSight Task Description (`customfield_10701`)
   - 업무 요약 (`customfield_10118`)
   - Task Description (`customfield_10162`)
   - 설계/구현/QA (`customfield_10142`)
2. **Task Screen Scheme 생성** — "AI Transform: Task Scheme"
   - Default Screen = "AI Transformation: Task Screen"
3. **Work Type Screen Scheme에서 매핑 추가**
   - Task → AI Transform: Task Scheme

---

## 7. 전체 설정 흐름 정리

### Screen 관련 계층 (아래→위로 읽기)

```
① Field (전역에 존재하는 개별 필드)
   예: SwiftSight Story Description, Due date, Assignee...

② Screen (필드들의 묶음 = "양식")
   예: "AI Transformation: Story Screen" = {Summary, Story Description, Priority...}

③ Screen Scheme (Create/Edit/View별 Screen 매핑 = "상황별 양식 선택")
   예: "AI Transform: Story Scheme" → Default = Story Screen

④ Work Type Screen Scheme (이슈 타입별 Screen Scheme 매핑 = "부서별 양식세트 배정")
   예: Story → Story Scheme, Task → Task Scheme

⑤ Project (프로젝트에 ④를 적용)
   예: AT 프로젝트 → "AT: Kanban Issue Type Screen Scheme"
```

### 실무 순서 (위→아래로 작업)

```
Step 1: Screen 생성 (필드 목록 정의)
  ↓
Step 2: Screen Scheme 생성 (Default Screen 지정)
  ↓
Step 3: Work Type Screen Scheme에서 이슈 타입별 매핑
  ↓
Step 4: (자동) 프로젝트에 적용됨 (이미 연결된 Scheme이면)
  ↓
Step 5: Custom Field Configuration에서 Context + Default Value 설정
  ↓
Step 6: Field Configuration에서 필드 Hidden 해제 (★ 빠뜨리기 쉬움)
  ↓
Step 7: Layout에서 필드 배치 순서 조정
```

---

## 8. Field Configuration — "Hidden" 필드 문제 해결

### 증상

Layout에서 필드를 추가했는데 **경고 삼각형(⚠️)**과 함께 메시지가 뜸:

> ⚠️ This field is hidden for 1 work type, and you can't see it in the work item.
> Go to "Default Field Configuration" to change this.

**필드가 Screen에는 있지만, 카드에서 보이지 않는 상태.**

### 원인: 이중 게이트

Jira에서 필드가 카드에 보이려면 **두 곳 모두** 통과해야 한다:

```
① Screen: 필드가 Screen에 포함되어 있는가? ✅
② Field Configuration: 필드가 Hidden이 아닌가? ← ❌ 여기서 막힘!
```

Screen에 필드를 추가해도, Field Configuration에서 해당 필드가 **Hidden** 상태면 카드에서 보이지 않는다.

```
필드가 카드에 보이는 조건:
  Screen에 포함 ✅  AND  Field Configuration에서 Hidden 아님 ✅
  → 둘 다 만족해야 보임

필드가 안 보이는 경우:
  Screen에 포함 ✅  AND  Field Configuration에서 Hidden ❌
  → Screen에 있어도 숨겨짐 (⚠️ 경고 표시)
```

### 해결 방법

```
경로: Jira Admin → Issues → Field Configurations → Default Field Configuration
URL: https://airsmed.atlassian.net/jira/settings/issues/field-configuration/10000
```

1. Field Configuration 목록에서 해당 필드를 찾기 (검색 활용)
2. 필드 옆의 **Hidden 토글**이 켜져 있으면 → **끄기 (Show)**
3. 또는 **"Add field"** 버튼으로 필드를 Configuration에 추가

### Field Configuration이란?

Field Configuration = **"필드별 동작 규칙"** 모음

각 필드에 대해 설정할 수 있는 것:

| 설정 | 설명 |
|------|------|
| **Required** | 필수 입력 여부 (ON/OFF) |
| **Hidden** | 숨김 여부 — **이게 ON이면 Screen에 있어도 안 보임** |
| **Renderer** | 렌더링 방식 (Wiki Style / Default Text) |
| **Screens** | 이 필드가 포함된 Screen 개수 (참조 정보) |

### Field Configuration Scheme

Field Configuration도 이슈 타입별로 다르게 적용할 수 있다:

```
Field Configuration Scheme (프로젝트에 적용)
├── Story → Story Field Configuration (Story용 필드 규칙)
├── Task  → Task Field Configuration (Task용 필드 규칙)
└── Default → Default Field Configuration (나머지)
```

AT 프로젝트는 현재 **"Default Field Configuration"** 하나만 사용하므로, 여기서 필드의 Hidden을 해제하면 모든 이슈 타입에 적용된다.

### 전체 흐름 정리 — 필드가 카드에 보이기까지

```
필드를 카드에 표시하려면 3가지를 모두 충족해야 한다:

1. Screen에 필드 포함          → Work Items → Screens
2. Field Configuration에서 Show → Work Items → Fields (또는 Jira Admin → Field Configurations)
3. Layout에서 배치             → Work Items → Layout

하나라도 빠지면:
- Screen 누락 → 필드 자체가 없음
- Field Config Hidden → ⚠️ 경고 + 안 보임
- Layout 미배치 → 숨겨진 상태 (접힌 섹션에 있을 수 있음)
```

### 트러블슈팅 체크리스트

필드가 카드에 안 보일 때:

```
□ Screen에 필드가 추가되어 있는가?
  → Project Settings → Screens → 해당 Screen 확인
  
□ Field Configuration에서 Hidden이 아닌가?
  → Project Settings → Fields → 해당 필드 확인
  → ⚠️ 경고가 있으면 "Go to Field Configuration" 클릭해서 해제
  
□ Layout에서 배치되어 있는가?
  → Project Settings → Layout → 해당 이슈 타입 확인
  
□ Custom Field Context가 이 프로젝트/이슈 타입에 적용되는가?
  → Jira Admin → Custom Fields → 해당 필드 → Configure → Context 확인
```
