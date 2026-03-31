# User Needs(요구사항) vs Use Case — 왜 분리해야 하는가

## 목차

- [1. 핵심 차이 한눈에 보기](#1-핵심-차이-한눈에-보기)
- [2. 각각이 답하는 질문](#2-각각이-답하는-질문)
- [3. 실제 예시로 비교](#3-실제-예시로-비교)
- [4. Use Case가 개발자에게 주는 가치](#4-use-case가-개발자에게-주는-가치)
- [5. PM에게 제안할 수 있는 구조](#5-pm에게-제안할-수-있는-구조)
- [6. 정리](#6-정리)
- [7. 현재 기획서(스토리카드)의 문제점 분석](#7-현재-기획서스토리카드의-문제점-분석)
- [8. 개선된 스토리카드 구조 제안](#8-개선된-스토리카드-구조-제안)
- [9. 설계 결정의 주체와 경계 — PM은 어디까지 쓰면 되는가](#9-설계-결정의-주체와-경계--pm은-어디까지-쓰면-되는가)
- [10. Solutions & Use Case 최종 개선안 — 역할 경계를 반영한 재작성](#10-solutions--use-case-최종-개선안--역할-경계를-반영한-재작성)

---

## 1. 핵심 차이 한눈에 보기

| 구분 | User Needs (요구사항) | Use Case |
|------|----------------------|----------|
| **관점** | **What** — 무엇을 원하는가 | **How** — 어떻게 사용하는가 |
| **초점** | 문제/목표 정의 | 시스템과 사용자의 **상호작용 흐름** |
| **주어** | 사용자/이해관계자 | 사용자 + **시스템** (둘 다 등장) |
| **추상 수준** | 높음 (의도, 기대효과) | 구체적 (step-by-step 시나리오) |
| **파생물** | 기능 목록, 우선순위 | **테스트 케이스**, QA 시나리오, 엣지 케이스 |
| **변경 빈도** | 비교적 안정적 | 구현 논의 중 자주 업데이트 |
| **독자** | PM, 경영진, 이해관계자 | **개발자, QA, 디자이너** |

---

## 2. 각각이 답하는 질문

### User Needs (요구사항)

> "**왜** 이걸 만들어야 하는가? **누가** 원하는가? **무엇이** 달라져야 하는가?"

```
- 누구: 정우진 (내부 요청자)
- 문제: Edit을 여러 번 하고 나중에 원할 때 Send하고 싶은데 현재 안 됨
- 기대효과: 과거 기록 조회 가능, 유저 니즈에 맞게 Edit/Approve 사용 가능
```

이것만으로는 **개발자가 어떤 동작을 구현해야 하는지 알 수 없습니다.**

### Use Case

> "사용자가 **구체적으로 어떤 순서로** 시스템과 상호작용하는가? **각 분기에서** 무슨 일이 일어나는가?"

```
- Auto send ON → report 자동 전송 → 이후 Review & Send 사용 가능
- Save만 하고 뒤로가기 → status 변하지 않음
- Save → Send → 다시 Edit → Save → 뒤로가기 → status 시계 모양 + 미전송 메시지
- Send to PACS 완료 → history에 전송 시간 업데이트 + PACS 아이콘
```

이것이 있어야 **테스트 케이스를 바로 뽑을 수 있습니다.**

---

## 3. 실제 예시로 비교

### 예시 A: 로그인 기능

#### 요구사항

```
- 누구: 모든 사용자
- 문제: 인증 없이 시스템에 접근할 수 있음
- 기대효과: 권한 있는 사용자만 접근 가능
- 해결책: ID/PW 기반 로그인 기능 구현
```

#### Use Case

```
UC-1: 정상 로그인
  1. 사용자가 로그인 페이지에 접속한다
  2. ID와 PW를 입력한다
  3. "로그인" 버튼을 클릭한다
  4. 시스템이 자격 증명을 검증한다
  5. 메인 대시보드로 리다이렉트된다

UC-2: 잘못된 비밀번호
  1~3. 동일
  4. 시스템이 자격 증명 검증 실패
  5. "비밀번호가 올바르지 않습니다" 에러 메시지 표시
  6. 입력 필드가 초기화되고 포커스가 PW 필드로 이동

UC-3: 5회 연속 실패
  1~4. UC-2 반복 5회
  5. 계정 잠금, "30분 후 재시도" 메시지 표시
  6. 관리자에게 알림 발송
```

> UC-2, UC-3은 요구사항만 보면 **절대 도출되지 않는** 시나리오입니다.

---

### 예시 B: Edit/Send 기능 (SwiftSight 스토리카드 기준)

#### 요구사항 (What)

```
- Edit과 Send를 분리하여 유저가 원할 때 전송할 수 있어야 함
- 과거 수정 기록을 열람할 수 있어야 함
- Auto send 설정이 가능해야 함
```

#### Use Case (How)

```
UC-1: Save만 하고 나가기
  1. 유저가 Report를 Edit한다
  2. "Save Changes"를 클릭한다
  3. 수정 내역이 반영된 보고서가 report viewer에 나타난다
  4. 뒤로가기를 한다
  5. report 탭의 status는 변하지 않는다 (시계 모양)

UC-2: Save 후 Send to PACS
  1~3. UC-1과 동일
  4. "Send to PACS"를 클릭한다
  5. 확인 모달이 표시된다 ("정말 보낼 것인지?")
  6. "OK"를 클릭한다
  7. 메인 페이지로 돌아오며 status가 "In Progress"로 변경
  8. Review & Send 버튼이 비활성화된다
  9. 전송 완료 시 우측 하단에 Chrome 알림 표시
  10. History에 전송 시간 업데이트 + PACS 아이콘 추가

UC-3: Send 후 재편집하고 Save만 한 경우
  1~9. UC-2 완료
  10. 다시 Edit → Save Changes
  11. 뒤로가기
  12. status가 시계 모양으로 변경
  13. 보고서 탭 상단에 "이 보고서는 아직 PACS에 전송되지 않았습니다" 메시지 표시

UC-4: Send 후 재편집하고 다시 Send
  1~10. UC-3과 동일
  11. "Send to PACS" 클릭
  12. 동일한 SeriesUID, Series Number로 PACS에 덮어쓰기

UC-5: Save 없이 Send 시도
  - Send to PACS 버튼이 비활성화 상태 → 클릭 불가

UC-6: Auto send ON 상태에서 최초 report 수신
  1. Patient report 수신
  2. 시스템이 자동으로 PACS 전송
  3. Review & Send 진입 시 History에 전송 시간 + PACS 아이콘 표시
  4. 이후 추가 Edit/Save/Send 가능 (UC-1~4와 동일)
```

---

## 4. Use Case가 개발자에게 주는 가치

### 4.1 테스트 케이스 직접 매핑

Use Case 하나가 곧 테스트 시나리오 하나입니다:

```typescript
// UC-3에서 바로 파생되는 테스트
describe('Send 후 재편집하고 Save만 한 경우', () => {
  it('status가 시계 모양으로 변경된다', () => { ... })
  it('상단에 미전송 메시지가 표시된다', () => { ... })
  it('Send to PACS 버튼이 비활성화된다', () => { ... })
})
```

요구사항만 있으면 이런 구체적 시나리오를 **개발자가 직접 상상해서 만들어야** 합니다.
그러면 PM이 의도한 것과 달라질 위험이 큽니다.

### 4.2 엣지 케이스 발견

Use Case를 작성하다 보면 자연스럽게 드러나는 질문들:

- "Save 없이 Send를 누르면?" → UC-5 도출
- "Send 중에 네트워크 끊기면?" → Fail 상태 시나리오 도출
- "History에 저장되는 보고서가 5개 초과하면?" → 삭제 정책 필요

이런 것들은 요구사항 레벨에서는 **보이지 않습니다.**

---

## 5. PM에게 제안할 수 있는 구조

합치는 대신, **계층적으로 연결**하는 방식을 추천합니다:

```
User Needs (요구사항)
├── N1: Edit과 Send 분리
│   ├── UC-1: Save만 하고 나가기
│   ├── UC-2: Save 후 Send to PACS
│   ├── UC-3: Send 후 재편집 + Save만
│   ├── UC-4: Send 후 재편집 + 재전송
│   └── UC-5: Save 없이 Send 시도
├── N2: 과거 기록 조회
│   ├── UC-6: History 탭에서 수정 내역 조회
│   └── UC-7: PACS 전송 기록 확인
└── N3: Auto send 설정
    ├── UC-8: Auto send ON 상태 최초 수신
    ├── UC-9: Auto send OFF 상태 최초 수신
    └── UC-10: General tab에서 설정 조회
```

**요구사항은 "왜/무엇"을 읽는 사람을 위해**, **Use Case는 "어떻게"를 구현하고 테스트하는 사람을 위해** 존재합니다.
합치면 두 목적 모두 흐려집니다.

---

## 6. 정리

### 합치면 잃는 것 vs 분리하면 얻는 것

| 합치면 잃는 것 | 분리하면 얻는 것 |
|---------------|-----------------|
| 비즈니스 의도가 구현 디테일에 묻힘 | 경영진은 요구사항만, 개발팀은 UC를 읽으면 됨 |
| 테스트 케이스 도출이 주관적 | UC → 테스트 1:1 매핑 가능 |
| 엣지 케이스가 누락됨 | UC 작성 과정에서 자연스럽게 발견 |
| 변경 영향도 파악 어려움 | "N1 변경 → UC-1~5 재검토" 추적 가능 |

### PM에게 드릴 한 마디

> "요구사항은 이해관계자와 합의하는 문서이고, Use Case는 개발팀이 정확히 같은 것을 만들도록 보장하는 문서입니다. 목적이 다르니 독자도 다릅니다."

---

## 7. 현재 기획서(스토리카드)의 문제점 분석

### 7.1 현재 스토리카드의 "Solutions" 섹션 원문

현재 PM이 작성한 Solutions 섹션은 다음과 같습니다:

> 1. Admin이 General tab에서 auto send 여부를 institution 별로 설정가능하게 한다
> 2. Edit 후에 save changes / send to pacs 버튼을 나눈다. save안 하고 뒤로가기를 하면, 보고서 뷰어창에서 아직 보고서가 PACS로 전송되지 않았습니다라는 메세지가 같이 뜬다.
> 3. Edit 상황에서 History탭을 누르면 이전 보고서 수정 내역들을 볼 수 있다.
> 4. Save changes를 누르면, 수정 내역이 반영된 상태의 보고서가 생성되어 report viewer에 나타나고, 리포트 뷰어창에서 수정 시간을 확인할 수 있다.
> 5. Send to PACS를 눌렀을 경우, 정말 보낼것인지 확인하는 modal창이 뜨고, okay를 누르면 메인페이지로 돌아오면서 patient report 탭 옆이 in progress상태가 된다. 이때 보고서는 report viewer에서 조회가 되나, review and send 버튼은 deactivate된다. (send to pacs가 끝날때까지)
> 6. Send to PACS가 완료되면 완료 알림창이 우측하단에 나타난다. (chrome 알림)
> 7. Send to pacs가 완료되면, 다시 review and send로 들어갔을때, history 내역의 수정시간이 pacs로 보낸시간으로 업데이트 되고, 우측에 PACS아이콘이 붙는다.
> 8. Auto send on/off의 차이는 최초로 patient report를 받았을때 바로 pacs로 보내냐, 유저의 click을 통해 pacs로 보내냐의 차이만 있다. 두 경우다 추가적인 edit/save/send to pacs가 가능하다.
> 9. auto send가 되었을 경우에는 review and send를 눌렀을때, 오른쪽 위 history에 pacs로 전송된 시간과 pacs아이콘이 같이있어야한다.
> 10. Auto send와 관련없이 pacs에 이미 보낸 보고서는 download/print가 가능하다.
> 11. patient report옆의 상태창은 Fail / In progress / Complete이 될 수 있다.

---

### 7.2 문제 진단: Solutions 섹션에 4가지 성격이 혼재

"Solutions"라는 단일 섹션 안에 **서로 다른 성격의 내용 4가지**가 뒤섞여 있습니다.

| # | PM이 쓴 내용 | 실제 성격 | 분류 근거 |
|---|-------------|----------|----------|
| ① | Admin이 General tab에서 auto send 여부를 institution 별로 설정가능하게 한다 | **Solution (설계 결정)** | "어떤 방식으로 해결할 것인가"에 대한 결정 |
| ② 앞부분 | Edit 후에 save changes / send to pacs 버튼을 나눈다 | **Solution (설계 결정)** | UI 구조에 대한 설계 결정 |
| ② 뒷부분 | save안 하고 뒤로가기를 하면, 보고서 뷰어창에서 "아직 보고서가 PACS로 전송되지 않았습니다"라는 메세지가 같이 뜬다 | **Use Case (조건부 흐름)** | "~하면 ~가 된다" — 사용자-시스템 상호작용 시나리오 |
| ③ | Edit 상황에서 History탭을 누르면 이전 보고서 수정 내역들을 볼 수 있다 | **기능 요구사항 (Requirement)** | "~할 수 있어야 한다" — 시스템이 제공해야 하는 능력 |
| ④ | Save changes를 누르면, 수정 내역이 반영된 상태의 보고서가 생성되어 report viewer에 나타나고... | **Use Case (상호작용 흐름)** | "~를 누르면 ~가 나타난다" — step-by-step 동작 기술 |
| ⑤ | Send to PACS를 눌렀을 경우, 정말 보낼것인지 확인하는 modal창이 뜨고, okay를 누르면... | **Use Case (상호작용 흐름)** | 여러 단계에 걸친 사용자-시스템 상호작용 시나리오 |
| ⑥ | Send to PACS가 완료되면 완료 알림창이 우측하단에 나타난다 (chrome 알림) | **UI Spec (UI 명세)** | 구체적인 UI 위치와 형태를 지정 |
| ⑦ | history 내역의 수정시간이 pacs로 보낸시간으로 업데이트 되고, 우측에 PACS아이콘이 붙는다 | **UI Spec + Use Case** | UI 요소의 시각적 표현 + 상태 변화 시나리오 혼합 |
| ⑧ | Auto send on/off의 차이는 최초로 patient report를 받았을때... | **Use Case (분기 설명)** | 두 시나리오 간의 분기 조건 및 흐름 설명 |
| ⑨ | auto send가 되었을 경우에는 review and send를 눌렀을때... | **UI Spec + Use Case** | 특정 조건에서의 UI 상태 명세 |
| ⑩ | Auto send와 관련없이 pacs에 이미 보낸 보고서는 download/print가 가능하다 | **기능 요구사항 (Requirement)** | "~가 가능해야 한다" — 시스템 능력 기술 |
| ⑪ | patient report옆의 상태창은 Fail / In progress / Complete이 될 수 있다 | **기능 요구사항 (Requirement)** | 상태값 정의 — 시스템 설계의 기본 규칙 |

시각적으로 정리하면:

```
현재 Solutions 섹션 (혼재 상태)
├── Solution (설계 결정)         → ①, ② 앞부분
├── 기능 요구사항 (Requirement)  → ③, ⑩, ⑪
├── Use Case (상호작용 흐름)     → ② 뒷부분, ④, ⑤, ⑧
└── UI Spec (UI 명세)           → ⑥, ⑦, ⑨
```

---

### 7.3 이 혼재가 만드는 구체적 문제

#### 문제 1: 개발자가 "이게 뭔지" 판단해야 하는 인지적 부담

개발자가 Solutions 섹션을 읽을 때 매 문장마다 스스로 분류해야 합니다:

- "이건 반드시 지켜야 할 요구사항인가?"
- "이건 PM의 제안인가, 확정된 결정인가?"
- "이건 디자이너와 합의된 UI 스펙인가?"
- "이건 내가 테스트해야 할 시나리오인가?"

**결과:** 같은 문서를 읽고도 개발자마다 다르게 해석할 위험이 있습니다.

#### 문제 2: QA가 테스트 케이스를 뽑으려면 문장을 재조합해야 함

현재 구조에서 "Save 후 Send to PACS" 시나리오를 테스트하려면:

- ④번에서 Save 동작을 읽고
- ⑤번에서 Send 동작을 읽고
- ⑥번에서 완료 알림을 읽고
- ⑦번에서 History 업데이트를 읽고
- ⑪번에서 상태값 정의를 참조해야 합니다

**하나의 시나리오를 이해하기 위해 5개의 흩어진 항목을 조합해야 합니다.**

#### 문제 3: 변경 영향도 추적 불가

"modal 대신 toast로 바꾸자"라는 변경이 발생했을 때:

- 이것이 요구사항 변경인가? → 아님
- 설계 결정 변경인가? → 아님
- UI 스펙 변경인가? → 맞음
- Use Case에 영향이 있는가? → ⑤번의 흐름이 바뀌므로 있음

현재 구조에서는 이 변경이 **어디에 영향을 미치는지 추적할 수 없습니다.**

#### 문제 4: Use Case 섹션과의 중복 및 혼동

현재 스토리카드에는 별도의 "Use Case" 섹션도 존재합니다. 그런데 Solutions 섹션에 이미 Use Case 성격의 내용(② 뒷부분, ④, ⑤, ⑧)이 포함되어 있어서:

- Solutions의 Use Case 내용과 Use Case 섹션의 내용이 **중복**됩니다
- 두 곳의 내용이 미묘하게 다를 경우 **어느 쪽이 정확한지** 판단할 수 없습니다
- PM이 "Use Case를 합치자"고 제안한 배경도 이 **중복 때문**으로 보입니다

---

### 7.4 Use Case 섹션 원문의 문제점 추가 분석

현재 Use Case 섹션도 살펴봅니다:

> - Auto send on → 유저의 승인 없이 physician report처럼 patient report를 자동으로 pacs에 전송
> - Auto send off - 유저의 Review & Send절차를 거쳐야만 pacs에 전송
> - Review 상황에서 save만하고 send to pacs를 안하고 뒤로가기 - report 탭의 status는 변하지 않음
> - Review 상황에서 save, send to pacs하고 다시 edit후 save까지하고 뒤로가기 - report탭의 status 시계모양으로 뜨고...
> - ...

#### Use Case 섹션 자체의 문제점

| 문제 | 설명 |
|------|------|
| **번호/ID 없음** | UC-1, UC-2 같은 식별자가 없어서 참조가 어려움 ("세 번째 항목의 그거"라고 말해야 함) |
| **Step-by-step이 아님** | "save만하고 send to pacs를 안하고 뒤로가기"처럼 한 문장에 여러 동작이 압축됨 |
| **사전조건(Precondition) 없음** | Auto send가 ON인지 OFF인지, 이전에 Send를 했는지 등의 시작 상태가 불명확 |
| **기대결과가 불완전** | "status 변하지 않음"은 있지만, 그 외 UI 요소(버튼 상태, 메시지 표시 등)의 기대결과 누락 |
| **기능 요구사항이 섞임** | "환자이름, 의사 정보를 수정할 수 있습니다", "저장개수는 최대 5개" 등은 Use Case가 아닌 요구사항 |
| **Solutions와 중복** | Solutions ④⑤와 Use Case의 내용이 같은 시나리오를 다른 표현으로 서술 |

---

## 8. 개선된 스토리카드 구조 제안

### 8.1 4-Layer 분리 구조

현재의 "Solutions에 다 넣기" 방식 대신, **역할별로 4개 레이어를 분리**합니다:

```
┌─────────────────────────────────────────────────────────┐
│  Layer 1: User Needs (요구사항) — WHY & WHAT             │
│  독자: 경영진, PM, 이해관계자                               │
│  질문: "왜 만드는가? 무엇이 달라져야 하는가?"                  │
│                                                         │
│  N1: Edit과 Send를 분리하고 싶다                           │
│  N2: 과거 수정 기록을 열람하고 싶다                          │
│  N3: 유저의 선호에 따라 Edit/Approve 절차를 생략하고 싶다      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 2: Solutions (설계 결정) — HOW TO SOLVE            │
│  독자: 아키텍트, 리드 개발자                                │
│  질문: "어떤 방식으로 해결할 것인가?"                         │
│                                                         │
│  S1: Save Changes / Send to PACS 버튼 분리 (N1 해결)     │
│  S2: History 탭 신설 (N2 해결)                            │
│  S3: Admin General tab에 auto send 토글 추가 (N3 해결)    │
│  S4: 상태값 정의: Fail / In Progress / Complete            │
│  S5: PACS 전송 보고서 저장 최대 5개 제한                    │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 3: Use Cases (사용 시나리오) — HOW USER INTERACTS   │
│  독자: 개발자, QA                                        │
│  질문: "사용자가 구체적으로 어떻게 상호작용하는가?"              │
│                                                         │
│  UC-1: Save만 하고 나가기                                 │
│  UC-2: Save 후 Send to PACS                             │
│  UC-3: Send 후 재편집 → Save만                           │
│  UC-4: Send 후 재편집 → 재전송                            │
│  UC-5: Save 없이 Send 시도                               │
│  UC-6: Auto send ON 최초 수신                             │
│  UC-7: Auto send OFF 최초 수신                            │
│  UC-8: History 탭에서 수정 내역 조회                        │
│  UC-9: 전송 실패 (Fail) 시나리오                           │
│  UC-10: PACS 전송 보고서가 5개 초과 시                      │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Layer 4: UI Spec (UI 명세) — HOW IT LOOKS               │
│  독자: 프론트엔드 개발자, 디자이너                           │
│  질문: "화면에 구체적으로 어떻게 보이는가?"                    │
│                                                         │
│  - 확인 모달: "정말 보낼 것인지?" + OK/Cancel 버튼          │
│  - 완료 알림: 우측 하단 Chrome notification                │
│  - History 항목: 수정 시간 표시 + PACS 아이콘 (전송 완료 시) │
│  - 미전송 배너: 보고서 뷰어 상단에 경고 메시지                 │
│  - 상태 아이콘: 시계 모양 (In Progress)                    │
│  - 비활성화 버튼: Review & Send (전송 중), Send (Save 전)   │
└─────────────────────────────────────────────────────────┘
```

---

### 8.2 개선된 스토리카드 작성 예시

아래는 현재 기획서의 내용을 4-Layer 구조로 재배치한 예시입니다.

---

#### Layer 1: User Needs (요구사항)

| ID | 요구사항 | 요청자 | 기대 효과 |
|----|---------|--------|----------|
| N1 | Edit을 여러 번 하고 나중에 원할 때 Send를 하고 싶다 (현재는 Approve와 동시에 Send) | 정우진 | 유저의 니즈에 맞게 Edit/Send 분리 사용 가능 |
| N2 | 재수정 시 예전 기록을 열람하고 싶다 (현재 불가) | 정우진 | 과거 보고서 수정 기록 조회 가능 |
| N3 | 유저의 선호에 따라 Edit/Approve 절차를 생략할 수 있어야 한다 | 정우진 | 워크플로우 유연성 확보 |

---

#### Layer 2: Solutions (설계 결정)

| ID | 설계 결정 | 해결하는 요구사항 |
|----|----------|-----------------|
| S1 | Edit 후 "Save Changes" / "Send to PACS" 두 개 버튼으로 분리 | N1 |
| S2 | Edit 상황에서 History 탭 신설, 이전 보고서 수정 내역 열람 가능 | N2 |
| S3 | Admin이 General tab에서 auto send 여부를 institution 별로 설정 가능 (default: ON) | N3 |
| S4 | Patient report 상태값: Fail / In Progress / Complete | N1, N3 |
| S5 | Send to PACS로 전송된 보고서 저장 개수 최대 5개 제한 | N2 |
| S6 | History 탭의 보고서 저장 단위는 "Save"를 기준으로 함 | N2 |
| S7 | 환자이름, 의사 정보 수정 가능 | N1 |
| S8 | General tab에서 user는 auto send 여부를 조회만 가능 (설정 변경은 Admin만) | N3 |

**상태값 정의 (S4 상세):**

| 상태 | 조건 | 예시 |
|------|------|------|
| **Fail** | 보고서가 PACS로 전송 실패된 경우 | 네트워크 오류, PACS 서버 다운 |
| **In Progress** | 가장 마지막 상태의 보고서가 PACS로 아직 안 보내졌을 때 | Save change가 마지막이고 Send to PACS 안 함 / Auto send OFF이고 아무런 동작 안 함 / Auto send ON인데, 그 이후 Save change를 하고 Send to PACS 안 함 |
| **Complete** | 가장 마지막 상태의 보고서가 PACS로 보내졌을 때 | Auto send ON이고 그 이후 아무런 동작 안 함 / Save change 후 Send to PACS를 누른 것이 마지막 동작 |

---

#### Layer 3: Use Cases (사용 시나리오)

##### UC-1: Save만 하고 나가기

- **사전조건:** 유저가 Report의 Edit 화면에 진입한 상태
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Report 내용을 수정한다 |
| 2 | 유저 | "Save Changes" 버튼을 클릭한다 |
| 3 | 시스템 | 수정 내역이 반영된 보고서가 생성되어 report viewer에 나타난다 |
| 4 | 시스템 | 리포트 뷰어에서 수정 시간을 표시한다 |
| 5 | 유저 | 뒤로가기를 한다 |
| 6 | 시스템 | report 탭의 status는 변하지 않는다 (시계 모양) |

- **기대결과:** 보고서는 저장되었으나 PACS에 전송되지 않음. Status = In Progress.

---

##### UC-2: Save 후 Send to PACS

- **사전조건:** UC-1의 Step 1~4 완료 (Save까지 된 상태)
- **관련 Solution:** S1, S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | "Send to PACS" 버튼을 클릭한다 |
| 2 | 시스템 | 확인 모달 표시: "정말 보낼 것인지?" (OK / Cancel) |
| 3 | 유저 | "OK"를 클릭한다 |
| 4 | 시스템 | 메인 페이지로 돌아오며 patient report 탭의 status가 "In Progress"로 변경 |
| 5 | 시스템 | 보고서는 report viewer에서 조회 가능하나, Review & Send 버튼은 비활성화 (전송 완료까지) |
| 6 | 시스템 | 전송 완료 시 우측 하단에 Chrome 알림 표시 |
| 7 | 시스템 | Status가 "Complete"로 변경 |
| 8 | 시스템 | History 내역의 수정 시간이 PACS 전송 시간으로 업데이트, PACS 아이콘 추가 |

- **기대결과:** 보고서가 PACS에 전송 완료. Status = Complete. History에 전송 기록 반영.

---

##### UC-3: Send 후 재편집 → Save만 하고 나가기

- **사전조건:** UC-2 완료 (PACS 전송 완료 상태)
- **관련 Solution:** S1, S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 다시 Edit 진입하여 내용을 수정한다 |
| 2 | 유저 | "Save Changes"를 클릭한다 |
| 3 | 시스템 | 수정 내역이 반영된 새 버전의 보고서가 생성된다 |
| 4 | 유저 | 뒤로가기를 한다 |
| 5 | 시스템 | report 탭의 status가 시계 모양 (In Progress)으로 변경 |
| 6 | 시스템 | 보고서 뷰어 상단에 "이 보고서는 아직 PACS에 전송되지 않았습니다" 메시지 표시 |

- **기대결과:** 최신 버전은 PACS 미전송. Status = In Progress. 미전송 경고 메시지 표시.

---

##### UC-4: Send 후 재편집 → 재전송

- **사전조건:** UC-3의 Step 1~3 완료 (재편집 후 Save된 상태)
- **관련 Solution:** S1, S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | "Send to PACS"를 클릭한다 |
| 2 | 시스템 | 확인 모달 표시 |
| 3 | 유저 | "OK"를 클릭한다 |
| 4 | 시스템 | 동일한 SeriesUID, Series Number로 PACS에 덮어쓰기 전송 |
| 5 | 시스템 | Status가 "Complete"로 변경, History 업데이트 |

- **기대결과:** PACS에 최신 보고서가 덮어쓰기됨. Status = Complete.
- **참고:** 덮어쓰기 불가능한 PACS는 현재 단계에서 고려하지 않음.

---

##### UC-5: Save 없이 Send 시도

- **사전조건:** Edit 진입했으나 Save Changes를 누르지 않은 상태
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | "Send to PACS" 버튼 클릭 시도 |
| 2 | 시스템 | 버튼이 비활성화 상태 → 클릭 불가 |

- **기대결과:** Send 기능은 Save 이후에만 활성화됨.

---

##### UC-6: Send to PACS 후 Save Changes 없이 다시 Send 시도

- **사전조건:** UC-2 완료 (PACS 전송 완료), 추가 Save Changes 없음
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 다시 Review & Send 진입 |
| 2 | 시스템 | "Send to PACS" 버튼이 비활성화 상태 |

- **기대결과:** 이미 전송된 보고서를 변경 없이 재전송할 수 없음. Save changes를 다시 하지 않으면 Send 기능 비활성화.

---

##### UC-7: Auto send ON — 최초 report 수신

- **사전조건:** Admin이 해당 institution에 auto send ON 설정 완료
- **관련 Solution:** S3, S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | Patient report 수신 |
| 2 | 시스템 | 유저의 승인 없이 자동으로 PACS에 전송 시작 |
| 3 | 시스템 | Status = "In Progress" |
| 4 | 시스템 | 전송 완료 시 Status = "Complete" |
| 5 | 유저 | Review & Send 진입 |
| 6 | 시스템 | History에 PACS 전송 시간 + PACS 아이콘 표시 |
| 7 | - | 이후 추가 Edit/Save/Send 가능 (UC-1~4와 동일하게 동작) |

- **기대결과:** 최초 보고서가 자동 전송됨. 이후 수동 Edit/Send 워크플로우 동일하게 적용.

---

##### UC-8: Auto send OFF — 최초 report 수신

- **사전조건:** Admin이 해당 institution에 auto send OFF 설정
- **관련 Solution:** S3, S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | Patient report 수신 |
| 2 | 시스템 | 자동 전송 없음. Status = "In Progress" |
| 3 | 유저 | Review & Send 절차를 거쳐야 PACS에 전송 (UC-1~2와 동일) |

- **기대결과:** 유저의 명시적 Send 동작이 있어야만 PACS 전송.

---

##### UC-9: History 탭에서 수정 내역 조회

- **사전조건:** 보고서에 1회 이상 Save가 발생한 상태
- **관련 Solution:** S2, S6

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Edit 상황에서 History 탭을 클릭한다 |
| 2 | 시스템 | 이전 보고서 수정 내역 목록 표시 (Save 기준으로 저장된 버전들) |
| 3 | 시스템 | 각 항목에 수정 시간 표시 |
| 4 | 시스템 | PACS에 전송된 버전에는 전송 시간 + PACS 아이콘 표시 |

- **기대결과:** 유저가 과거 수정 이력을 시간순으로 확인 가능.

---

##### UC-10: PACS 전송 완료 보고서 Download/Print

- **사전조건:** PACS에 전송 완료된 보고서가 존재
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 전송 완료된 보고서를 조회한다 |
| 2 | 시스템 | Download / Print 기능 활성화 |
| 3 | 유저 | Download 또는 Print를 선택한다 |

- **기대결과:** Auto send 여부와 관계없이, PACS에 전송된 보고서는 Download/Print 가능.

---

##### UC-11: 전송 실패 (Fail) 시나리오

- **사전조건:** Send to PACS 실행 중 오류 발생
- **관련 Solution:** S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | PACS 전송 중 오류 발생 (네트워크, 서버 등) |
| 2 | 시스템 | Status = "Fail"로 변경 |
| 3 | 시스템 | 실패 알림 표시 |

- **기대결과:** 유저가 전송 실패 상태를 인지할 수 있음.
- **미정의 사항:** Fail 후 재시도 절차 (자동 재시도? 수동 재전송? 재시도 횟수 제한?)

---

##### UC-12: General tab에서 Auto send 설정 조회 (일반 유저)

- **사전조건:** 일반 유저 (Admin이 아닌)가 General tab에 접근
- **관련 Solution:** S3, S8

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | General tab을 연다 |
| 2 | 시스템 | Auto send 설정 현황을 표시 (조회만 가능, 변경 불가) |

- **기대결과:** 일반 유저는 auto send 여부를 확인만 할 수 있음.

---

#### Layer 4: UI Spec (UI 명세)

| ID | UI 요소 | 위치 | 상세 |
|----|---------|------|------|
| UI-1 | Save Changes 버튼 | Edit 화면 하단 | 수정 내용을 저장 |
| UI-2 | Send to PACS 버튼 | Edit 화면 하단 (Save 옆) | Save 후에만 활성화. Send 후 Save changes 없이 재클릭 불가 |
| UI-3 | 확인 모달 | 화면 중앙 | "정말 보낼 것인지?" + OK / Cancel |
| UI-4 | 완료 알림 | 우측 하단 | Chrome notification. Send to PACS 완료 시 표시 |
| UI-5 | 미전송 배너 | 보고서 뷰어 상단 | "이 보고서는 아직 PACS에 전송되지 않았습니다" |
| UI-6 | History 탭 | Edit 화면 우측 상단 | 수정 내역 목록. Save 기준으로 버전 관리 |
| UI-7 | History 항목 — 수정 시간 | History 탭 내 각 항목 | Save 시점의 시간 표시 |
| UI-8 | History 항목 — PACS 아이콘 | History 탭 내 각 항목 우측 | PACS 전송 완료된 버전에만 표시. 전송 시간으로 업데이트 |
| UI-9 | Status 아이콘 | Patient report 탭 옆 | 시계 모양 (In Progress) / 체크 (Complete) / X (Fail) |
| UI-10 | Auto send 토글 | Admin > General tab | Institution 별 ON/OFF 설정 (default: ON) |
| UI-11 | Auto send 조회 | User > General tab | 현재 설정값 표시 (조회만 가능, 변경 불가) |
| UI-12 | Download/Print 버튼 | Report viewer | PACS 전송 완료된 보고서에서만 활성화 |

---

### 8.3 역할별 읽는 방법

이 구조에서 각 역할은 자신에게 필요한 레이어만 읽으면 됩니다:

| 역할 | 주로 읽는 레이어 | 목적 |
|------|----------------|------|
| 경영진 / 이해관계자 | **Layer 1** (User Needs) | "왜 이 기능을 만드는가" 이해 |
| PM | **Layer 1 + 2** (User Needs + Solutions) | 요구사항과 설계 결정 관리 |
| 아키텍트 / 리드 개발자 | **Layer 2** (Solutions) | 기술적 설계 결정 검토 |
| 개발자 | **Layer 2 + 3** (Solutions + Use Cases) | 구현해야 할 동작의 정확한 흐름 파악 |
| QA | **Layer 3** (Use Cases) | 테스트 케이스 도출 (UC → TC 1:1 매핑) |
| 프론트엔드 개발자 | **Layer 3 + 4** (Use Cases + UI Spec) | 화면 구현 + 상호작용 흐름 |
| 디자이너 | **Layer 4** (UI Spec) | UI 요소의 위치, 형태, 상태 확인 |

---

### 8.4 Traceability (추적성) 매트릭스

4-Layer 구조의 가장 큰 장점은 **변경 발생 시 영향 범위를 즉시 파악**할 수 있다는 것입니다:

| 변경 사항 | 영향받는 Layer | 영향받는 항목 |
|----------|--------------|-------------|
| "Auto send 기능 삭제" | Layer 1 | N3 삭제 |
| | Layer 2 | S3, S8 삭제 |
| | Layer 3 | UC-7, UC-8, UC-12 삭제 |
| | Layer 4 | UI-10, UI-11 삭제 |
| "Modal 대신 Toast로 변경" | Layer 3 | UC-2 Step 2~3 수정 |
| | Layer 4 | UI-3 수정 |
| "저장 개수 5개 → 10개" | Layer 2 | S5 수정 |
| | Layer 3 | 영향 없음 (로직 동일) |

---

### 8.5 현재 기획서에서 누락된 시나리오 (Use Case 작성을 통해 발견)

Use Case를 체계적으로 작성하는 과정에서 현재 기획서에 **명시되지 않은 시나리오**들이 발견됩니다:

| # | 누락된 시나리오 | PM에게 확인 필요한 질문 |
|---|---------------|---------------------|
| 1 | **Fail 후 재시도** | 자동 재시도가 있는가? 수동 재전송 버튼이 있는가? 재시도 횟수 제한은? |
| 2 | **Edit 중 Save 없이 뒤로가기** | 수정 내용이 사라지는가? "저장하지 않은 변경사항이 있습니다" 경고가 필요한가? |
| 3 | **동시 편집** | 두 명의 유저가 같은 보고서를 동시에 Edit하면? 나중에 Save한 사람이 덮어쓰는가? |
| 4 | **History 5개 초과 시** | 가장 오래된 것부터 삭제? PACS에 전송된 것은 보존? 유저에게 알림? |
| 5 | **Auto send ON인데 전송 실패** | 유저에게 알림이 가는가? 자동 재시도가 있는가? 유저가 수동 재전송 가능한가? |
| 6 | **Send to PACS 중 유저가 Edit 진입** | 전송 중에 Edit가 가능한가? 잠금(lock)이 걸리는가? |
| 7 | **Cancel 클릭 (확인 모달)** | 모달이 닫히고 Edit 화면으로 돌아가는가? 다른 동작이 있는가? |
| 8 | **PACS에 덮어쓰기 불가 시** | 현재 고려 안 한다고 했지만, 에러 핸들링은 어떻게 하는가? |
| 9 | **환자이름/의사정보 수정** | 수정 시 이미 PACS에 전송된 보고서에도 반영되는가? 새로 전송해야 하는가? |

> 이러한 누락 시나리오들은 **요구사항만으로는 발견할 수 없고**, Use Case를 step-by-step으로 작성해야 비로소 드러납니다. 이것이 Use Case를 요구사항과 분리하여 유지해야 하는 가장 실질적인 이유입니다.

---

## 9. 설계 결정의 주체와 경계 — PM은 어디까지 쓰면 되는가

### 9.1 "설계 결정"에도 레벨이 있다

"Solutions(설계 결정)"이라는 단어 하나로 뭉뚱그리면 **PM이 모든 설계를 한다**는 오해가 생깁니다.
실제로 설계 결정은 3개 레벨로 나뉘며, 각 레벨의 주체가 다릅니다:

| 레벨 | 이름 | 결정 주체 | 핵심 질문 | 예시 |
|------|------|----------|----------|------|
| **Level 1** | Product-level 설계 | **PM** | "어떤 기능으로 문제를 해결할 것인가?" | "Save와 Send를 분리한다", "Auto send 토글을 제공한다" |
| **Level 2** | Technical-level 설계 | **개발자** | "기술적으로 어떻게 구현할 것인가?" | "상태 관리를 state machine으로 한다", "PACS 전송은 비동기 큐로 처리한다" |
| **Level 3** | UI-level 설계 | **디자이너** | "화면에 어떤 형태로 보여줄 것인가?" | "확인 모달의 레이아웃", "알림의 위치와 애니메이션", "PACS 아이콘의 디자인" |

시각적으로:

```
        PM의 영역                개발자의 영역              디자이너의 영역
  ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
  │  Product-level   │     │  Technical-level │     │    UI-level     │
  │                 │     │                 │     │                 │
  │  "무엇을 만들까"  │ ──→ │  "어떻게 구현할까" │ ──→ │ "어떻게 보여줄까" │
  │                 │     │                 │     │                 │
  │  기능 방향 결정   │     │  아키텍처/로직    │     │  시각/인터랙션    │
  └─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

### 9.2 PM이 현재 기획서에서 월권한 부분

현재 Solutions 섹션의 각 항목을 다시 보면, PM이 자기 영역을 넘어선 부분이 있습니다:

| # | PM이 쓴 내용 | 레벨 | 원래 주체 | 문제 |
|---|-------------|------|----------|------|
| ① | Admin이 General tab에서 auto send 여부를 institution 별로 설정가능하게 한다 | **L1 (Product)** | PM | **적절함** — "auto send 기능을 제공한다"는 PM의 결정 |
| ② | Save changes / send to pacs 버튼을 나눈다 | **L1 (Product)** | PM | **적절함** — "두 기능을 분리한다"는 PM의 결정 |
| ⑤ | **확인하는 modal창이 뜨고** | **L3 (UI)** | 디자이너 | **월권** — "확인 절차가 필요하다"까지가 PM, "modal"은 디자이너의 결정 |
| ⑥ | **완료 알림창이 우측하단에 나타난다 (chrome 알림)** | **L3 (UI)** | 디자이너 | **월권** — "완료 알림이 필요하다"까지가 PM, "우측 하단 + Chrome 알림"은 디자이너의 결정 |
| ⑦ | **우측에 PACS아이콘이 붙는다** | **L3 (UI)** | 디자이너 | **월권** — "전송 완료 여부를 구분할 수 있어야 한다"까지가 PM, "아이콘의 위치와 형태"는 디자이너의 결정 |
| ⑨ | **오른쪽 위 history에** pacs로 전송된 시간과 pacs아이콘이 같이있어야한다 | **L3 (UI)** | 디자이너 | **월권** — 위치 지정("오른쪽 위")은 디자이너 영역 |
| ⑪ | **상태창은 Fail / In progress / Complete** | **L1 (Product)** | PM | **적절함** — 상태값 정의는 비즈니스 규칙 |

---

### 9.3 PM이 쓰면 충분한 것 vs 쓰면 안 되는 것

#### PM이 써야 하는 것 (Product-level)

PM은 **"무엇이 필요한가"**와 **"왜 필요한가"**를 씁니다.
**"어떻게 보이는가"**와 **"어떻게 동작하는가"**는 쓰지 않습니다.

| PM이 쓰면 충분한 표현 | PM이 쓰면 월권인 표현 |
|---------------------|---------------------|
| "Send 전에 **확인 절차**가 필요하다" | "**modal창**이 뜨고 **OK 버튼**을 누르면..." |
| "전송 완료 시 **유저에게 알림**이 가야 한다" | "**우측 하단에 Chrome 알림**이 나타난다" |
| "PACS 전송 완료 여부를 **구분할 수 있어야** 한다" | "**우측에 PACS 아이콘**이 붙는다" |
| "전송 이력을 **시간순으로 확인**할 수 있어야 한다" | "**오른쪽 위 History**에 수정시간이..." |
| "Save와 Send를 **분리**한다" | "Save 버튼은 **하단 왼쪽**, Send는 **하단 오른쪽**에..." |
| "전송 중에는 **재전송을 방지**해야 한다" | "Review & Send 버튼이 **회색으로 비활성화**된다" |
| "보고서 상태는 3가지로 구분한다" | "상태 아이콘은 **시계 모양** / **체크** / **X**이다" |

**핵심 원칙:**

```
PM의 문장 = "~할 수 있어야 한다" / "~가 필요하다" / "~를 제공한다"
디자이너의 문장 = "~에 위치한다" / "~모양이다" / "~색상이다" / "~애니메이션으로"
개발자의 문장 = "~방식으로 처리한다" / "~패턴으로 구현한다" / "~큐를 사용한다"
```

---

### 9.4 PM이 월권하면 생기는 문제

#### 문제 1: 디자이너의 전문성이 무시된다

PM이 "modal로 확인한다"고 쓰면, 디자이너는 **더 나은 대안을 제안할 기회를 잃습니다.**

```
PM이 "modal"이라고 못 박으면:
  → 디자이너가 "이 맥락에서는 inline confirmation이 더 적합합니다"라고
    말하기 어려워짐
  → 이미 기획서에 "modal"이라고 적혀있으므로

PM이 "확인 절차가 필요하다"라고만 쓰면:
  → 디자이너가 context에 맞는 최적의 확인 방식을 제안할 수 있음
  → modal, toast, inline confirmation, bottom sheet 등 다양한 옵션 검토 가능
```

#### 문제 2: 개발자가 기술적 제약을 반영할 수 없다

PM이 "Chrome 알림"이라고 쓰면, 개발자는 **기술적 한계를 논의하기 어려워집니다.**

```
PM이 "Chrome 알림"이라고 못 박으면:
  → 개발자: "Chrome Notification API는 유저가 권한을 거부할 수 있는데..."
  → 이미 기획서에 적혀있으므로 "기획대로 안 되는데요"라는 대화가 됨

PM이 "전송 완료 시 유저에게 알림이 가야 한다"라고만 쓰면:
  → 개발자 + 디자이너가 함께: "in-app notification이 더 안정적입니다"
  → 기술적 실현 가능성을 고려한 최선의 방식을 팀이 결정
```

#### 문제 3: 변경 시 PM이 병목이 된다

PM이 UI까지 기획서에 적으면, **UI 변경마다 기획서를 수정해야** 합니다.

```
현재: "우측 하단에 Chrome 알림" → 디자이너가 "좌측 상단 toast로 변경"
  → PM이 기획서 수정해야 함 → 리뷰 → 승인 → 개발 시작

분리 후: PM 기획서에는 "완료 알림이 필요하다"만 있음
  → 디자이너가 UI 스펙에서 "좌측 상단 toast"로 변경
  → PM 기획서 수정 불필요 → 바로 개발 진행
```

---

### 9.5 현재 기획서의 Solutions를 PM 영역으로만 재작성

현재 PM이 쓴 11개 항목을 **PM이 쓰면 충분한 수준**으로 다시 쓰면 이렇게 됩니다:

#### Before (현재 — PM이 모든 레벨을 다 씀)

```
① Admin이 General tab에서 auto send 여부를 institution 별로 설정가능하게 한다
② Edit 후에 save changes / send to pacs 버튼을 나눈다. save안 하고 뒤로가기를 하면,
   보고서 뷰어창에서 아직 보고서가 PACS로 전송되지 않았습니다라는 메세지가 같이 뜬다.
③ Edit 상황에서 History탭을 누르면 이전 보고서 수정 내역들을 볼 수 있다.
④ Save changes를 누르면, 수정 내역이 반영된 상태의 보고서가 생성되어
   report viewer에 나타나고, 리포트 뷰어창에서 수정 시간을 확인할 수 있다.
⑤ Send to PACS를 눌렀을 경우, 정말 보낼것인지 확인하는 modal창이 뜨고,
   okay를 누르면 메인페이지로 돌아오면서 patient report 탭 옆이
   in progress상태가 된다...
⑥ Send to PACS가 완료되면 완료 알림창이 우측하단에 나타난다. (chrome 알림)
⑦ history 내역의 수정시간이 pacs로 보낸시간으로 업데이트 되고,
   우측에 PACS아이콘이 붙는다.
⑧ Auto send on/off의 차이는 최초로 patient report를 받았을때 바로 pacs로
   보내냐, 유저의 click을 통해 pacs로 보내냐의 차이만 있다.
⑨ auto send가 되었을 경우에는 review and send를 눌렀을때,
   오른쪽 위 history에 pacs로 전송된 시간과 pacs아이콘이 같이있어야한다.
⑩ Auto send와 관련없이 pacs에 이미 보낸 보고서는 download/print가 가능하다.
⑪ patient report옆의 상태창은 Fail / In progress / Complete이 될 수 있다.
```

#### After (개선 — PM은 Product-level만 씀)

```
Solutions (PM 작성 — Product-level 설계 결정만)

S1. Edit과 Send 기능을 분리한다.
    - 유저가 보고서를 수정(Save)한 후, 별도의 동작으로 PACS에 전송(Send)할 수 있어야 한다.
    - Save 없이는 Send할 수 없다.
    - Send 후 추가 Save 없이는 재전송할 수 없다.

S2. 보고서 수정 이력을 조회할 수 있어야 한다.
    - 이력의 저장 단위는 Save 기준이다.
    - PACS에 전송된 이력은 전송 여부와 전송 시간을 구분할 수 있어야 한다.
    - PACS 전송 보고서의 저장 개수는 최대 5개로 제한한다.

S3. Auto send 기능을 제공한다.
    - Admin이 institution 별로 auto send 여부를 설정할 수 있다. (default: ON)
    - 일반 유저는 auto send 설정을 조회만 할 수 있다.
    - Auto send ON: 최초 report 수신 시 유저 승인 없이 자동 전송.
    - Auto send OFF: 유저의 명시적 Send 동작이 있어야 전송.
    - Auto send 여부와 관계없이, 이후 추가 Edit/Save/Send는 동일하게 동작한다.

S4. 보고서 전송 상태를 3가지로 관리한다.
    - Fail: 전송 실패
    - In Progress: 최신 보고서가 아직 PACS에 전송되지 않음
    - Complete: 최신 보고서가 PACS에 전송 완료

S5. Send 전에 유저에게 확인 절차를 거친다.

S6. Send 완료 시 유저에게 알림을 제공한다.

S7. 전송 중에는 재전송/재편집 관련 기능을 비활성화한다.

S8. PACS에 전송된 보고서는 Download/Print가 가능하다.

S9. 환자이름, 의사 정보를 수정할 수 있다.

S10. 재전송 시 동일한 SeriesUID, Series Number로 PACS에 덮어쓰기한다.
     (덮어쓰기 불가 PACS는 현재 단계에서 미고려)
```

#### 비교: Before vs After

| 관점 | Before | After |
|------|--------|-------|
| **분량** | 11개 항목, UI/흐름/요구사항 혼재 | 10개 항목, Product-level 결정만 |
| **"modal"** | PM이 "modal"로 지정 | "확인 절차를 거친다"만 명시 → 디자이너가 최적 방식 결정 |
| **"우측 하단 Chrome 알림"** | PM이 위치+방식 지정 | "알림을 제공한다"만 명시 → 디자이너+개발자가 최적 방식 결정 |
| **"PACS 아이콘"** | PM이 UI 요소 지정 | "전송 여부를 구분할 수 있어야"만 명시 → 디자이너가 표현 방식 결정 |
| **Use Case 혼입** | ②④⑤⑧에 상호작용 흐름 포함 | 흐름 설명 없음 → Use Case 섹션으로 분리 |
| **디자이너 자율성** | 낮음 (이미 PM이 UI 결정) | 높음 (PM은 "무엇"만, "어떻게 보이는가"는 디자이너) |
| **개발자 자율성** | 낮음 (구현 방식까지 암시) | 높음 (PM은 "무엇"만, "어떻게 구현하는가"는 개발자) |

---

### 9.6 각 역할이 채워야 할 설계 영역

PM이 Product-level만 쓰면, 나머지는 **디자이너와 개발자가 각자의 전문 영역에서 채웁니다:**

#### PM이 쓴 후 → 디자이너가 채우는 것

PM의 "S5. Send 전에 유저에게 확인 절차를 거친다"를 받아서:

```
[디자이너의 UI Spec]

UI-3: Send 확인 절차
  - 방식: Modal dialog (전체 화면 dim 처리)
  - 위치: 화면 중앙
  - 메시지: "보고서를 PACS에 전송하시겠습니까?"
  - 버튼: "전송" (Primary, Blue) / "취소" (Secondary, Gray)
  - 접근성: ESC 키로 닫기 가능, 포커스 트랩 적용

  [대안 검토]
  - Inline confirmation: 기각 — 중요한 외부 전송이므로 주의 환기 필요
  - Toast: 기각 — 유저가 놓칠 수 있음
  - Bottom sheet: 기각 — 데스크탑 앱이므로 modal이 더 적합
```

PM의 "S6. Send 완료 시 유저에게 알림을 제공한다"를 받아서:

```
[디자이너의 UI Spec]

UI-4: 전송 완료 알림
  - 방식: In-app toast notification
  - 위치: 우측 하단
  - 지속 시간: 5초 후 자동 소멸
  - 아이콘: 체크마크 (Green)
  - 메시지: "보고서가 PACS에 성공적으로 전송되었습니다"
  - 클릭 시: 해당 보고서의 Review & Send 화면으로 이동

  [Chrome Notification 미사용 사유]
  - 유저가 브라우저 알림 권한을 거부할 수 있음
  - 앱 내에서 발생한 이벤트이므로 in-app 알림이 더 적절
```

#### PM이 쓴 후 → 개발자가 채우는 것

PM의 "S4. 보고서 전송 상태를 3가지로 관리한다"를 받아서:

```
[개발자의 Technical Design]

상태 관리: State Machine 패턴 적용

  States: IDLE → IN_PROGRESS → COMPLETE | FAIL

  Transitions:
    IDLE → IN_PROGRESS     : save() 호출 시
    IN_PROGRESS → COMPLETE : sendToPacs() 성공 시
    IN_PROGRESS → FAIL     : sendToPacs() 실패 시
    COMPLETE → IN_PROGRESS : 재편집 후 save() 시
    FAIL → IN_PROGRESS     : 재시도 시

  PACS 전송: 비동기 큐 (Bull Queue + Redis)
    - Retry: 최대 3회, exponential backoff
    - Timeout: 30초
    - Dead letter queue: 3회 실패 시 DLQ로 이동
```

PM의 "S10. 재전송 시 동일한 SeriesUID로 덮어쓰기"를 받아서:

```
[개발자의 Technical Design]

DICOM 전송 전략:
  - C-STORE SCP로 전송
  - 동일 SeriesUID + SeriesNumber 유지
  - StudyInstanceUID는 변경하지 않음
  - 덮어쓰기 미지원 PACS 감지 시: 에러 코드 반환 + Fail 상태 전환
```

---

### 9.7 PM이 지켜야 할 작성 규칙 요약

#### 쓰면 되는 것 (Product-level)

| 카테고리 | PM이 쓰는 내용 | 작성 패턴 |
|---------|--------------|----------|
| **기능 결정** | 어떤 기능을 만들 것인가 | "~기능을 제공한다" |
| **비즈니스 규칙** | 상태값, 제한, 조건 | "상태는 A/B/C이다", "최대 N개" |
| **유저 권한** | 누가 무엇을 할 수 있는가 | "Admin만 ~가능", "유저는 조회만 가능" |
| **기능 간 관계** | 기능의 선후 관계, 의존성 | "Save 없이는 Send 불가" |
| **Scope 결정** | 현재 단계에서 고려할 것/안 할 것 | "덮어쓰기 불가 PACS는 현재 미고려" |

#### 쓰면 안 되는 것 (다른 역할의 영역)

| 카테고리 | 쓰면 안 되는 예시 | 대신 이렇게 써야 함 | 결정 주체 |
|---------|-----------------|-------------------|----------|
| **UI 위치** | "우측 하단에", "오른쪽 위에" | (쓰지 않음) | 디자이너 |
| **UI 형태** | "modal창", "아이콘", "시계 모양" | "확인 절차", "구분할 수 있어야" | 디자이너 |
| **UI 방식** | "Chrome 알림", "toast" | "알림을 제공한다" | 디자이너 |
| **상호작용 흐름** | "~누르면 ~뜨고 ~누르면 ~된다" | (Use Case 섹션에 분리) | PM + 개발자 협업 |
| **구현 방식** | "비동기로 처리", "큐를 사용" | (쓰지 않음) | 개발자 |
| **데이터 구조** | "SeriesUID 필드에 저장" | "동일 식별자로 덮어쓰기" | 개발자 |

#### 판단 기준 한 줄

> **PM의 문장에 "시각적 형태"나 "위치"나 "기술 용어"가 들어가면, 그건 다른 사람의 영역을 침범한 것이다.**

---

### 9.8 실무에서의 협업 흐름

PM이 Product-level만 쓴 뒤, 실제 프로젝트에서는 이렇게 흘러갑니다:

```
Step 1: PM이 User Needs + Solutions (Product-level) 작성
        ↓
Step 2: 킥오프 미팅 — PM이 발표, 개발자/디자이너가 질문
        ↓
Step 3: 디자이너가 UI Spec 작성 (피그마 + 문서)
        개발자가 Technical Design 작성 (필요 시)
        PM + 개발자가 Use Case 협업 작성
        ↓
Step 4: 리뷰 — 각 영역의 전문가가 교차 검토
        PM: Use Case의 비즈니스 로직 검토
        개발자: Use Case의 기술적 실현 가능성 검토
        디자이너: UI Spec의 일관성/접근성 검토
        ↓
Step 5: 확정 → 개발 시작
```

이 흐름에서 **PM의 기획서가 끝나는 시점은 Step 1**입니다.
Step 3~4에서 나오는 Use Case, UI Spec, Technical Design은 **각 전문가의 문서**입니다.

PM이 혼자서 Step 1~3을 다 쓰려고 하면:
- 디자이너/개발자의 전문성이 반영되지 않고
- PM이 병목이 되며
- 변경 시마다 PM 기획서를 고쳐야 합니다

---

### 9.9 정리: 각 역할의 문서 책임

| 문서 | 작성자 | 검토자 | PM이 쓸 분량 |
|------|--------|--------|-------------|
| **User Needs** | PM | 이해관계자, 경영진 | PM이 **100%** 작성 |
| **Solutions (Product-level)** | PM | 개발자, 디자이너 | PM이 **100%** 작성 |
| **Use Cases** | PM + 개발자 **협업** | QA | PM이 **초안 50%**, 개발자가 **엣지케이스 50%** 보완 |
| **UI Spec** | 디자이너 | PM, 프론트엔드 개발자 | PM은 **0%** (디자이너 영역) |
| **Technical Design** | 개발자 | 아키텍트, 리드 개발자 | PM은 **0%** (개발자 영역) |

> **PM에게 드릴 한 마디:** "기획서에 'modal', '우측 하단', 'Chrome 알림' 같은 단어가 들어가는 순간, 그것은 디자이너의 결정을 대신 내린 것입니다. '확인 절차가 필요하다', '알림이 필요하다'만 쓰세요. 어떻게 보여줄지는 디자이너가, 어떻게 구현할지는 개발자가 결정합니다."

---

## 10. Solutions & Use Case 최종 개선안 — 역할 경계를 반영한 재작성

Section 8에서는 기존 기획서의 내용을 4-Layer로 **재배치**했습니다.
Section 9에서는 PM이 **어디까지만 써야 하는지** 경계를 정의했습니다.

이 Section 10에서는 두 기준을 모두 적용하여, **실제로 스토리카드에 들어가야 할 최종 형태**를 제시합니다.

---

### 10.1 Section 8의 개선안에 아직 남아있던 문제

Section 8.2에서 작성한 개선안에도 Section 9의 기준을 적용하면 **여전히 PM 월권 표현이 남아있습니다:**

#### Solutions (Layer 2)에서의 문제

| Section 8의 표현 | 문제 | 개선 방향 |
|-----------------|------|----------|
| S1: Edit 후 **"Save Changes" / "Send to PACS" 두 개 버튼으로** 분리 | **버튼 레이블**은 디자이너 영역 | "Save와 Send 기능을 분리한다" |
| S2: Edit 상황에서 **History 탭** 신설 | **"탭"이라는 UI 형태**는 디자이너 영역 | "보고서 수정 이력을 조회할 수 있어야 한다" |
| S3: Admin이 **General tab에서** auto send 여부를 설정 | **위치 지정**은 디자이너 영역 | "Admin이 institution 별로 auto send를 설정할 수 있다" |
| S8: **General tab에서** user는 조회만 가능 | **위치 지정**은 디자이너 영역 | "일반 유저는 auto send 설정을 조회만 할 수 있다" |

#### Use Cases (Layer 3)에서의 문제

| Section 8의 표현 | 문제 | 개선 방향 |
|-----------------|------|----------|
| UC-1 Step 6: status는 변하지 않는다 **(시계 모양)** | **아이콘 형태**는 디자이너 영역 | "status는 변하지 않는다 (In Progress 유지)" |
| UC-2 Step 2: **확인 모달** 표시 | **"모달"**은 디자이너 영역 | "확인 절차가 표시된다" |
| UC-2 Step 6: **우측 하단에 Chrome 알림** 표시 | **위치+방식**은 디자이너 영역 | "전송 완료 알림이 표시된다" |
| UC-2 Step 8: **PACS 아이콘** 추가 | **아이콘**은 디자이너 영역 | "PACS 전송 완료 여부가 구분되어 표시된다" |
| UC-3 Step 5: **시계 모양** (In Progress) | **아이콘 형태**는 디자이너 영역 | "Status = In Progress로 변경" |
| UC-4 Step 2: **확인 모달** 표시 | **"모달"**은 디자이너 영역 | "확인 절차가 표시된다" |
| UC-7 Step 6: **PACS 전송 시간 + PACS 아이콘** 표시 | **아이콘**은 디자이너 영역 | "PACS 전송 이력이 구분되어 표시된다" |
| UC-9 Step 4: **PACS에 전송된 버전에는 전송 시간 + PACS 아이콘 표시** | **아이콘**은 디자이너 영역 | "전송 완료 여부와 전송 시간이 구분되어 표시된다" |

---

### 10.2 최종 개선안: Solutions (PM이 작성하는 Product-level만)

```
Solutions (PM 작성)

S1. Edit과 Send 기능을 분리한다.
    - 유저가 보고서를 수정(Save)한 후, 별도의 동작으로 PACS에 전송(Send)할 수 있어야 한다.
    - Save 없이는 Send할 수 없다.
    - Send 후 추가 Save 없이는 재전송할 수 없다.

S2. 보고서 수정 이력을 조회할 수 있어야 한다.
    - 이력의 저장 단위는 Save 기준이다.
    - PACS에 전송된 이력은 전송 여부와 전송 시간을 구분할 수 있어야 한다.
    - PACS 전송 보고서의 저장 개수는 최대 5개로 제한한다.

S3. Auto send 기능을 제공한다.
    - Admin이 institution 별로 auto send 여부를 설정할 수 있다. (default: ON)
    - 일반 유저는 auto send 설정을 조회만 할 수 있다.
    - Auto send ON: 최초 report 수신 시 유저 승인 없이 자동 전송.
    - Auto send OFF: 유저의 명시적 Send 동작이 있어야 전송.
    - Auto send 여부와 관계없이, 이후 추가 Edit/Save/Send는 동일하게 동작한다.

S4. 보고서 전송 상태를 3가지로 관리한다.
    - Fail: 전송 실패
    - In Progress: 최신 보고서가 아직 PACS에 전송되지 않음
    - Complete: 최신 보고서가 PACS에 전송 완료

S5. Send 전에 유저에게 확인 절차를 거친다.

S6. Send 완료 시 유저에게 알림을 제공한다.

S7. 전송 중에는 재전송 관련 기능을 비활성화한다.

S8. PACS에 전송된 보고서는 Download/Print가 가능하다.

S9. 환자이름, 의사 정보를 수정할 수 있다.

S10. 재전송 시 동일한 식별자로 PACS에 덮어쓰기한다.
     (덮어쓰기 불가 PACS는 현재 단계에서 미고려)

S11. 최신 보고서가 PACS에 미전송 상태일 경우, 유저에게 미전송 상태임을 안내한다.
```

> **포인트:** "modal", "Chrome 알림", "아이콘", "탭", "General tab", "버튼 레이블" 등 UI 표현이 **하나도 없습니다.** PM은 **기능과 규칙만** 정의합니다.

---

### 10.3 최종 개선안: Use Cases (PM + 개발자 협업 작성)

Use Case도 마찬가지로 **UI 구현 방식을 언급하지 않고**, 사용자와 시스템의 **행동(Behavior)**만 기술합니다.
디자이너가 결정할 "어떻게 보여주는가"는 `→ UI Spec 참조` 태그로 연결합니다.

---

#### UC-1: Save만 하고 나가기

- **사전조건:** 유저가 보고서의 편집 화면에 진입한 상태
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 보고서 내용을 수정한다 |
| 2 | 유저 | 저장(Save) 동작을 실행한다 |
| 3 | 시스템 | 수정 내역이 반영된 보고서가 생성된다 |
| 4 | 시스템 | 수정 시간이 기록된다 |
| 5 | 유저 | 편집 화면을 나간다 |
| 6 | 시스템 | 보고서 상태는 변하지 않는다 (In Progress 유지) |

- **기대결과:** 보고서는 저장되었으나 PACS에 전송되지 않음. Status = In Progress.

---

#### UC-2: Save 후 Send to PACS

- **사전조건:** UC-1의 Step 1~4 완료 (Save까지 된 상태)
- **관련 Solution:** S1, S4, S5, S6

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 전송(Send) 동작을 실행한다 |
| 2 | 시스템 | 유저에게 전송 확인 절차를 제시한다 `→ UI Spec: 확인 절차의 형태` |
| 3 | 유저 | 전송을 승인한다 |
| 4 | 시스템 | PACS 전송을 시작한다. Status = "In Progress" |
| 5 | 시스템 | 전송 중에는 재전송 관련 기능을 비활성화한다 `→ UI Spec: 비활성화 표현 방식` |
| 6 | 시스템 | 전송 완료 시 유저에게 완료 알림을 제공한다 `→ UI Spec: 알림 형태/위치` |
| 7 | 시스템 | Status = "Complete"로 변경 |
| 8 | 시스템 | 수정 이력에 PACS 전송 시간이 기록되고, 전송 완료 여부가 구분되어 표시된다 `→ UI Spec: 전송 완료 구분 표현` |

- **기대결과:** 보고서가 PACS에 전송 완료. Status = Complete. 수정 이력에 전송 기록 반영.

---

#### UC-3: Send 후 재편집 → Save만 하고 나가기

- **사전조건:** UC-2 완료 (PACS 전송 완료 상태)
- **관련 Solution:** S1, S4, S11

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 다시 편집 화면에 진입하여 내용을 수정한다 |
| 2 | 유저 | 저장(Save) 동작을 실행한다 |
| 3 | 시스템 | 수정 내역이 반영된 새 버전의 보고서가 생성된다 |
| 4 | 유저 | 편집 화면을 나간다 |
| 5 | 시스템 | Status = "In Progress"로 변경 |
| 6 | 시스템 | 최신 보고서가 PACS에 미전송 상태임을 유저에게 안내한다 `→ UI Spec: 미전송 안내 표현` |

- **기대결과:** 최신 버전은 PACS 미전송. Status = In Progress. 미전송 상태 안내.

---

#### UC-4: Send 후 재편집 → 재전송

- **사전조건:** UC-3의 Step 1~3 완료 (재편집 후 Save된 상태)
- **관련 Solution:** S1, S4, S5, S10

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 전송(Send) 동작을 실행한다 |
| 2 | 시스템 | 유저에게 전송 확인 절차를 제시한다 `→ UI Spec: 확인 절차의 형태` |
| 3 | 유저 | 전송을 승인한다 |
| 4 | 시스템 | 동일한 식별자로 PACS에 덮어쓰기 전송한다 |
| 5 | 시스템 | Status = "Complete"로 변경, 수정 이력 업데이트 |

- **기대결과:** PACS에 최신 보고서가 덮어쓰기됨. Status = Complete.
- **참고:** 덮어쓰기 불가능한 PACS는 현재 단계에서 고려하지 않음.

---

#### UC-5: Save 없이 Send 시도

- **사전조건:** 편집 진입했으나 Save를 하지 않은 상태
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 전송(Send) 동작을 시도한다 |
| 2 | 시스템 | Send 기능이 비활성화 상태이므로 실행 불가 `→ UI Spec: 비활성화 표현 방식` |

- **기대결과:** Send 기능은 Save 이후에만 활성화됨.

---

#### UC-6: Send to PACS 후 Save 없이 다시 Send 시도

- **사전조건:** UC-2 완료 (PACS 전송 완료), 추가 Save 없음
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 보고서 Review 화면에 진입한다 |
| 2 | 시스템 | Send 기능이 비활성화 상태 (변경 사항 없으므로) `→ UI Spec: 비활성화 표현 방식` |

- **기대결과:** 이미 전송된 보고서를 변경 없이 재전송할 수 없음.

---

#### UC-7: Auto send ON — 최초 report 수신

- **사전조건:** Admin이 해당 institution에 auto send ON 설정 완료
- **관련 Solution:** S3, S4, S6

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | Patient report를 수신한다 |
| 2 | 시스템 | 유저의 승인 없이 자동으로 PACS에 전송을 시작한다. Status = "In Progress" |
| 3 | 시스템 | 전송 완료 시 유저에게 완료 알림을 제공한다 `→ UI Spec: 알림 형태/위치` |
| 4 | 시스템 | Status = "Complete"로 변경 |
| 5 | 유저 | 보고서 Review 화면에 진입한다 |
| 6 | 시스템 | 수정 이력에 PACS 전송 기록이 구분되어 표시된다 `→ UI Spec: 전송 완료 구분 표현` |
| 7 | - | 이후 추가 Edit/Save/Send 가능 (UC-1~4와 동일하게 동작) |

- **기대결과:** 최초 보고서가 자동 전송됨. 이후 수동 Edit/Send 워크플로우 동일하게 적용.

---

#### UC-8: Auto send OFF — 최초 report 수신

- **사전조건:** Admin이 해당 institution에 auto send OFF 설정
- **관련 Solution:** S3, S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | Patient report를 수신한다 |
| 2 | 시스템 | 자동 전송 없음. Status = "In Progress" |
| 3 | 유저 | Review & Send 절차를 거쳐야 PACS에 전송 가능 (UC-1~2와 동일) |

- **기대결과:** 유저의 명시적 Send 동작이 있어야만 PACS 전송.

---

#### UC-9: 수정 이력 조회

- **사전조건:** 보고서에 1회 이상 Save가 발생한 상태
- **관련 Solution:** S2

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 편집 화면에서 수정 이력 조회 기능에 접근한다 `→ UI Spec: 이력 조회 UI 형태` |
| 2 | 시스템 | 이전 보고서 수정 내역 목록을 표시한다 (Save 기준으로 저장된 버전들) |
| 3 | 시스템 | 각 항목에 수정 시간을 표시한다 |
| 4 | 시스템 | PACS에 전송된 버전은 전송 완료 여부와 전송 시간이 구분되어 표시된다 `→ UI Spec: 전송 완료 구분 표현` |

- **기대결과:** 유저가 과거 수정 이력을 시간순으로 확인 가능. PACS 전송 여부 구분 가능.

---

#### UC-10: PACS 전송 완료 보고서 Download/Print

- **사전조건:** PACS에 전송 완료된 보고서가 존재
- **관련 Solution:** S8

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 전송 완료된 보고서를 조회한다 |
| 2 | 시스템 | Download / Print 기능이 활성화된다 `→ UI Spec: 활성화 표현 방식` |
| 3 | 유저 | Download 또는 Print를 실행한다 |

- **기대결과:** Auto send 여부와 관계없이, PACS에 전송된 보고서는 Download/Print 가능.

---

#### UC-11: 전송 실패 (Fail) 시나리오

- **사전조건:** Send to PACS 실행 중 오류 발생
- **관련 Solution:** S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | PACS 전송 중 오류 발생 (네트워크, 서버 등) |
| 2 | 시스템 | Status = "Fail"로 변경 |
| 3 | 시스템 | 유저에게 전송 실패를 알린다 `→ UI Spec: 실패 알림 형태` |

- **기대결과:** 유저가 전송 실패 상태를 인지할 수 있음.
- **미정의 사항:** Fail 후 재시도 절차 (PM 결정 필요)

---

#### UC-12: Auto send 설정 조회 (일반 유저)

- **사전조건:** 일반 유저 (Admin이 아닌)가 설정 화면에 접근
- **관련 Solution:** S3

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | 설정 화면에 접근한다 `→ UI Spec: 설정 화면 위치` |
| 2 | 시스템 | Auto send 설정 현황을 표시한다 (조회만 가능, 변경 불가) `→ UI Spec: 읽기 전용 표현` |

- **기대결과:** 일반 유저는 auto send 여부를 확인만 할 수 있음.

---

### 10.4 최종 개선안: UI Spec (디자이너가 작성)

Use Case에서 `→ UI Spec` 태그로 연결된 항목들을 디자이너가 채웁니다.
PM은 이 섹션을 **작성하지 않습니다.**

```
[디자이너 작성]

UI Spec: 확인 절차의 형태 (UC-2, UC-4에서 참조)
  - 방식: Modal dialog
  - 위치: 화면 중앙, 배경 dim 처리
  - 메시지: "보고서를 PACS에 전송하시겠습니까?"
  - 버튼: "전송" (Primary) / "취소" (Secondary)
  - 접근성: ESC 키로 닫기, 포커스 트랩

UI Spec: 알림 형태/위치 (UC-2, UC-7에서 참조)
  - 방식: In-app toast notification
  - 위치: 우측 하단
  - 지속 시간: 5초 후 자동 소멸
  - 메시지: "보고서가 PACS에 성공적으로 전송되었습니다"

UI Spec: 전송 완료 구분 표현 (UC-2, UC-7, UC-9에서 참조)
  - 전송 완료 버전: PACS 아이콘 + 전송 시간 표시
  - 미전송 버전: 아이콘 없음, 수정 시간만 표시

UI Spec: 비활성화 표현 방식 (UC-2, UC-5, UC-6에서 참조)
  - 비활성화 버튼: 회색 처리 + 클릭 불가 + tooltip "저장 후 전송 가능"

UI Spec: 미전송 안내 표현 (UC-3에서 참조)
  - 방식: 보고서 뷰어 상단 배너
  - 메시지: "이 보고서는 아직 PACS에 전송되지 않았습니다"
  - 스타일: Warning (Yellow background)

UI Spec: 이력 조회 UI 형태 (UC-9에서 참조)
  - 방식: 편집 화면 우측 사이드패널 (History 탭)
  - 정렬: 최신순
  - 각 항목: 수정 시간 + (전송 완료 시) PACS 아이콘

UI Spec: 보고서 상태 표현 (UC 전반에서 참조)
  - In Progress: 시계 아이콘 (Gray)
  - Complete: 체크 아이콘 (Green)
  - Fail: X 아이콘 (Red)
  - 위치: Patient report 탭 옆

UI Spec: 설정 화면 (UC-12에서 참조)
  - Admin: General tab > Auto send 토글 (ON/OFF)
  - User: General tab > Auto send 상태 표시 (읽기 전용, 토글 비활성화)

UI Spec: Download/Print (UC-10에서 참조)
  - 위치: Report viewer 상단
  - 조건: PACS 전송 완료 보고서에서만 활성화

UI Spec: 실패 알림 형태 (UC-11에서 참조)
  - 방식: In-app toast notification
  - 스타일: Error (Red)
  - 메시지: "보고서 전송에 실패했습니다. 다시 시도해주세요."
```

---

### 10.5 Before vs After 전체 비교

#### Solutions 비교

| 항목 | Section 8 (1차 개선) | Section 10 (최종 개선) | 변경 이유 |
|------|---------------------|----------------------|----------|
| S1 | "Save Changes" / "Send to PACS" **두 개 버튼으로** 분리 | Edit과 Send **기능을** 분리한다 | 버튼 레이블/형태는 디자이너 영역 |
| S2 | **History 탭** 신설, 이전 보고서 수정 내역 열람 가능 | 보고서 수정 이력을 **조회할 수 있어야** 한다 | "탭"이라는 UI 형태 제거 |
| S3 | Admin이 **General tab에서** auto send 여부를 설정 | Admin이 **institution 별로** auto send를 설정할 수 있다 | 위치 지정 제거 |
| S5 | (없었음) | Send 전에 유저에게 **확인 절차**를 거친다 | "modal"이 아닌 행동 수준 기술 |
| S6 | (없었음) | Send 완료 시 유저에게 **알림을 제공**한다 | "Chrome 알림"이 아닌 행동 수준 기술 |
| S11 | (없었음) | 미전송 상태일 경우 유저에게 **안내**한다 | "배너"가 아닌 행동 수준 기술 |

#### Use Case 비교 (UC-2 예시)

| Step | Section 8 (1차 개선) | Section 10 (최종 개선) | 변경 이유 |
|------|---------------------|----------------------|----------|
| 1 | "Send to PACS" **버튼을 클릭**한다 | 전송(Send) **동작을 실행**한다 | "버튼 클릭"은 UI 구현 방식 |
| 2 | **확인 모달** 표시: "정말 보낼 것인지?" | 유저에게 전송 **확인 절차를 제시**한다 `→ UI Spec` | "모달"은 디자이너 결정 |
| 3 | **"OK"를 클릭**한다 | 전송을 **승인**한다 | "OK 클릭"은 UI 구현 방식 |
| 5 | **Review & Send 버튼은 비활성화** | 재전송 관련 기능을 **비활성화**한다 `→ UI Spec` | 특정 버튼명은 디자이너 영역 |
| 6 | **우측 하단에 Chrome 알림** 표시 | 유저에게 **완료 알림을 제공**한다 `→ UI Spec` | 위치+방식은 디자이너 영역 |
| 8 | **PACS 아이콘** 추가 | 전송 완료 여부가 **구분되어 표시**된다 `→ UI Spec` | 아이콘은 디자이너 영역 |

---

### 10.6 `→ UI Spec` 태그 패턴 — Use Case와 UI Spec의 연결 방식

최종 개선안에서 도입한 `→ UI Spec` 태그의 핵심:

```
[Use Case — 행동을 기술]
  "유저에게 전송 확인 절차를 제시한다"     → UI Spec: 확인 절차의 형태
  "유저에게 완료 알림을 제공한다"          → UI Spec: 알림 형태/위치
  "전송 완료 여부가 구분되어 표시된다"      → UI Spec: 전송 완료 구분 표현
  "기능이 비활성화된다"                   → UI Spec: 비활성화 표현 방식

[UI Spec — 디자이너가 구체적 형태를 결정]
  확인 절차의 형태      → Modal dialog, 화면 중앙, OK/Cancel
  알림 형태/위치        → Toast, 우측 하단, 5초 후 자동 소멸
  전송 완료 구분 표현   → PACS 아이콘 + 전송 시간
  비활성화 표현 방식    → 회색 처리 + tooltip
```

이 방식의 장점:

| 장점 | 설명 |
|------|------|
| **PM은 행동만 씀** | "확인 절차를 제시한다" — what, not how |
| **디자이너는 형태만 씀** | "Modal, 화면 중앙" — how it looks |
| **변경이 독립적** | Modal → Toast로 바꿔도 Use Case 수정 불필요 |
| **재사용 가능** | "비활성화 표현 방식"은 UC-2, UC-5, UC-6에서 공유 |
| **테스트 분리** | QA는 Use Case로 기능 테스트, 디자이너는 UI Spec으로 시각 테스트 |

---

### 10.7 각 역할이 작성하는 최종 범위

```
┌──────────────────────────────────────────────────────────────┐
│                     PM이 작성하는 영역                         │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  User Needs: "왜, 누구를 위해, 무엇을"                    │  │
│  │  → N1, N2, N3                                         │  │
│  └────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Solutions: "어떤 기능으로 해결할 것인가" (Product-level)   │  │
│  │  → S1~S11 (UI 표현, 구현 방식 없이!)                     │  │
│  └────────────────────────────────────────────────────────┘  │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Use Case 초안: 주요 흐름(Happy path)                    │  │
│  │  → UC-1~UC-4의 기본 시나리오                             │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
┌─────────────────────────┐  ┌─────────────────────────┐
│   개발자가 보완하는 영역    │  │   디자이너가 작성하는 영역  │
│                         │  │                         │
│  Use Case 보완:          │  │  UI Spec:               │
│  → 엣지 케이스 추가       │  │  → 확인 절차의 형태       │
│    (UC-5, UC-6, UC-11)  │  │  → 알림 형태/위치         │
│  → 기술적 제약 반영       │  │  → 상태 아이콘 디자인      │
│  → 에러 시나리오          │  │  → 비활성화 표현          │
│                         │  │  → 미전송 안내 표현        │
│  Technical Design:       │  │  → 레이아웃/색상/간격      │
│  → 상태 관리 방식         │  │                         │
│  → PACS 전송 아키텍처     │  │  피그마:                 │
│  → 에러 핸들링 전략       │  │  → 화면 목업              │
│  → 데이터 모델           │  │  → 인터랙션 프로토타입      │
└─────────────────────────┘  └─────────────────────────┘
```

---

### 10.8 정리: 한 눈에 보는 개선 흐름

```
원본 기획서 (PM이 모든 것을 다 씀)
  ↓
[Section 7] 문제 진단: 4가지 성격 혼재 발견
  ↓
[Section 8] 1차 개선: 4-Layer로 분리 (구조는 좋으나 표현에 UI 월권 잔존)
  ↓
[Section 9] 기준 수립: PM은 Product-level만, 디자이너/개발자 영역 정의
  ↓
[Section 10] 최종 개선: 역할 경계를 완전히 반영한 재작성
  - Solutions: "기능/규칙만" (UI 표현 제거)
  - Use Cases: "행동만" (UI 구현 방식을 → UI Spec 태그로 분리)
  - UI Spec: 디자이너가 채움 (PM은 0% 작성)
```

> **핵심:** PM의 기획서에서 **"modal", "우측 하단", "Chrome 알림", "PACS 아이콘", "시계 모양", "History 탭", "General tab"**이 모두 사라지고, 대신 **"확인 절차", "알림 제공", "전송 완료 구분", "이력 조회", "설정 관리"**라는 행동 수준의 표현만 남습니다. 구체적인 형태와 위치는 디자이너가, 구현 방식은 개발자가 각자의 전문 영역에서 결정합니다.
