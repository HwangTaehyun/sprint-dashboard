# SS-371: SwiftSight Job 처리 상태 가시성 개선

---

## User Needs

누구의 어떤 요구사항/문제를 해결하려고 하나요?

**내부 요청자:** (요청자명)

### N1: Job 처리 진행률을 실시간으로 확인하고 싶다 (우선순위: 중상)

**구체적 문제점:** 현재 SwiftSight에서 Report 처리가 어느 정도 진행되었는지 알 수 없다. SM(Study Manager)에서는 Progress를 확인할 수 있는 반면, SwiftSight에서는 처리 중인지 완료되었는지만 알 수 있어 대기 시간에 대한 예측이 불가능하다.

**기대 효과:** Report별 처리 진행 상황을 실시간으로 파악하여 대기 시간 예측 및 업무 효율성 향상.

### N2: Job 실행 시 각 스텝별 처리 시간과 에러 정보를 상세히 알고 싶다 (우선순위: 상)

**구체적 문제점:** Job이 실패하거나 지연될 때, 어느 스텝에서 문제가 발생했는지, 어떤 에러인지 파악할 수 없다. 문제 원인을 찾기 위해 별도의 로그 조회나 개발팀 문의가 필요하다.

**기대 효과:**
- 각 스텝별 처리 시간을 확인하여 병목 구간을 파악할 수 있다.
- 에러 발생 시 즉시 원인을 파악하여 대응 시간을 단축할 수 있다.
- 개발팀 문의 없이 자체적으로 문제를 진단할 수 있다.

### N3: Mismatching 발생 시 원인을 파악하고 싶다 (우선순위: 상)

**구체적 문제점:** Series 간 Mismatching이 발생했을 때 왜 Mismatching이 발생했는지 알 수 없다. 어떤 시리즈가 어떤 이유로 매칭되지 않았는지 정보가 제공되지 않는다.

**기대 효과:** Mismatching 원인을 즉시 파악하여 데이터 정합성 문제를 빠르게 해결할 수 있다.

---

## Solutions

어떤 기능을 통해 문제를 해결할 것인지 간략하게 설명해주세요.

**해결책 유형:** ✅ Improvement

### S1. Report별 처리 진행률을 스텝 기반 퍼센트로 표시한다. (N1 해결)

- 각 Report의 전체 처리 과정 대비 현재 진행 상황을 유저가 확인할 수 있어야 한다.
- 진행률은 실시간 또는 준실시간으로 업데이트되어야 한다.
- **진행률 산정 방식:** Report 처리에 대한 남은 시간 표기는 불가 (개발팀 검토 결과). 대신 **각 Step 단위로 퍼센트를 균등 분배**한다.
- **Job Log Steps (5단계, 각 20%):**

| Step | 이름 | 진행률 |
|------|------|--------|
| 1 | Processing Start | 20% |
| 2 | Upload End Time | 40% |
| 3 | Modal Analysis Time | 60% |
| 4 | Report Generation Time | 80% |
| 5 | Store non-review reports | 100% |

- Step 5 "Store non-review reports"는 Review가 필요하지 않는 Report들만 자동 Store 했다는 의미이다.

### S2. Job Detail에서 Job Logs와 User Action Logs를 표시한다. (N2 해결)

- **Job Logs:** Job의 각 처리 스텝별로 소요 시간을 확인할 수 있어야 한다.
  - 스텝 목록: Processing Start → Upload End Time → Modal Analysis Time → Report Generation Time → Store non-review reports
- **User Action Logs:** (기존 "Report Logs"에서 명칭 변경) Report에 대한 사용자 액션 이력을 보여준다.
  - 표시 대상 액션: Report Edit, Report Approve, Send to PACS 등의 User Action들
- 참고: SM의 Job Detail Info 표시 방식을 참고한다.

### S3. 각 스텝별 에러 발생 위치와 에러 내용을 표시한다. (N2 해결)

- 에러 발생 시 어느 스텝에서 발생했는지 식별할 수 있어야 한다.
- 에러의 구체적인 내용(에러 메시지)을 확인할 수 있어야 한다.
- 표시 대상 에러 유형:
    - PACS 전송 에러 메시지
    - Upload / Download 시 timeout 에러
    - 기타 처리 중 발생하는 에러
- 에러 상세 정보는 지속적으로 확인 가능해야 한다 (일시적/hover 표시가 아닌, 화면에 상시 노출). `→ UI Spec: 에러 상세 표현 방식`

### S4. Mismatching 발생 시 원인 정보를 Notification으로 안내한다. (N3 해결)

- Mismatching이 발생하면 어떤 시리즈에서 발생했는지 알 수 있어야 한다.
- 시리즈별 series name 정보를 포함하여 안내한다.
- 안내 방식은 기존 Notification 기능을 활용한다.
- (개발 feasibility 검토 결과, 별도 UI 신설 대신 Notification 활용으로 결정)

---

## Use Cases

### UC-1: Report 처리 진행률 확인

- **사전조건:** 1건 이상의 Report가 처리 중인 상태
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Report 목록을 조회한다 |
| 2 | 시스템 | 처리 중인 Report에 대해 현재 진행률을 표시한다 `→ UI Spec: 진행률 표현 방식` |
| 3 | 시스템 | 처리가 진행됨에 따라 진행률이 업데이트된다 |
| 4 | 시스템 | 처리 완료 시 100% / 완료 상태로 전환된다 |

- **기대결과:** 유저가 각 Report의 처리 진행 상황을 실시간으로 파악할 수 있음.

---

### UC-2: Job Detail에서 스텝별 처리 시간 확인

- **사전조건:** 처리 완료(성공 또는 실패)된 Job이 존재
- **관련 Solution:** S2

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Job Detail 화면에 접근한다 |
| 2 | 시스템 | Job의 각 처리 스텝 목록을 표시한다 |
| 3 | 시스템 | 각 스텝별 소요 시간을 표시한다 `→ UI Spec: 스텝별 시간 표현 방식` |

- **기대결과:** 유저가 어느 스텝에서 시간이 오래 걸렸는지 파악 가능.

---

### UC-3: 에러 발생 시 스텝별 에러 상세 확인

- **사전조건:** Job 처리 중 에러가 발생한 상태
- **관련 Solution:** S2, S3

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Job Detail 화면에 접근한다 |
| 2 | 시스템 | 각 스텝의 처리 결과(성공/실패)를 표시한다 |
| 3 | 시스템 | 에러가 발생한 스텝을 식별할 수 있도록 표시한다 `→ UI Spec: 에러 스텝 구분 표현` |
| 4 | 시스템 | 에러가 발생한 스텝의 에러 메시지를 상시 노출한다 `→ UI Spec: 에러 상세 표현 방식` |

- **기대결과:** 유저가 어느 스텝에서 어떤 에러가 발생했는지 즉시 파악 가능. 별도 조작(hover 등) 없이 에러 내용이 지속적으로 보임.

---

### UC-4: PACS 전송 에러 확인

- **사전조건:** PACS 전송 스텝에서 에러 발생
- **관련 Solution:** S3

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Job Detail 화면에 접근한다 |
| 2 | 시스템 | PACS 전송 스텝이 실패 상태로 표시된다 |
| 3 | 시스템 | PACS 전송 에러 메시지가 상시 노출된다 `→ UI Spec: 에러 상세 표현 방식` |

- **기대결과:** PACS 측에서 반환한 에러 메시지를 유저가 직접 확인 가능.

---

### UC-5: Upload/Download Timeout 에러 확인

- **사전조건:** Upload 또는 Download 스텝에서 timeout 에러 발생
- **관련 Solution:** S3

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Job Detail 화면에 접근한다 |
| 2 | 시스템 | Upload 또는 Download 스텝이 실패 상태로 표시된다 |
| 3 | 시스템 | Timeout 에러임을 식별할 수 있는 에러 메시지가 상시 노출된다 `→ UI Spec: 에러 상세 표현 방식` |

- **기대결과:** Timeout이 발생한 스텝과 에러 유형을 유저가 직접 확인 가능.

---

### UC-6: 정상 완료된 Job의 스텝별 정보 확인

- **사전조건:** Job이 정상적으로 완료된 상태
- **관련 Solution:** S2

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 유저 | Job Detail 화면에 접근한다 |
| 2 | 시스템 | 모든 스텝이 성공 상태로 표시된다 |
| 3 | 시스템 | 각 스텝별 소요 시간이 표시된다 |

- **기대결과:** 정상 완료된 경우에도 각 스텝의 처리 시간을 확인할 수 있어, 성능 모니터링에 활용 가능.

---

### UC-7: Mismatching 발생 시 Notification 수신

- **사전조건:** Series 간 Mismatching이 발생
- **관련 Solution:** S4

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | Series 간 Mismatching을 감지한다 |
| 2 | 시스템 | Notification을 생성한다 (시리즈별 series name 포함) |
| 3 | 유저 | Notification을 확인한다 |
| 4 | 시스템 | Mismatching이 발생한 시리즈 정보와 series name을 표시한다 |

- **기대결과:** 유저가 어떤 시리즈에서 Mismatching이 발생했는지 Notification을 통해 즉시 파악 가능.

---

### UC-8: 처리 중인 Report의 진행률 변화 확인

- **사전조건:** Report가 처리 중이며 유저가 Report 목록 화면에 있는 상태
- **관련 Solution:** S1

| Step | Actor | Action / System Response |
|------|-------|--------------------------|
| 1 | 시스템 | 처리가 진행됨에 따라 진행률 정보가 업데이트된다 |
| 2 | 유저 | 화면에서 진행률 변화를 확인한다 |
| 3 | 시스템 | 처리 완료 시 완료 상태로 전환된다 |

- **기대결과:** 유저가 페이지 새로고침 없이(또는 최소한의 조작으로) 진행 상황의 변화를 확인 가능.

---

## 기능 요구사항

| ID | 요구사항 | 관련 Solution | 우선순위 |
|----|---------|--------------|---------|
| FR-1 | Report별 처리 진행률을 스텝 기반 퍼센트(20%씩 5단계)로 표시한다 | S1 | 중상 |
| FR-2 | Job Detail에서 Job Logs — 각 스텝별 처리 시간을 표시한다 (Processing Start / Upload End Time / Modal Analysis Time / Report Generation Time / Store non-review reports) | S2 | 상 |
| FR-2-1 | Job Detail에서 User Action Logs — Report Edit, Approve, Send to PACS 등 사용자 액션 이력을 표시한다 | S2 | 상 |
| FR-3 | 에러 발생 스텝을 식별할 수 있어야 한다 | S3 | 상 |
| FR-4 | 에러 메시지를 상시 노출한다 (hover 방식 아님) | S3 | 상 |
| FR-5 | PACS 전송 에러 메시지를 표시한다 | S3 | 상 |
| FR-6 | Upload/Download timeout 에러를 표시한다 | S3 | 상 |
| FR-7 | Mismatching 발생 시 series name을 포함한 Notification을 생성한다 | S4 | 상 |

---

## 참고 사항

- **SM(Study Manager) 참고:** Progress 표시 방식, Job Detail Info의 스텝별 처리 시간 표현 방식을 참고한다.
- **Mismatching UI 관련:** 별도 UI 신설은 개발 feasibility 검토 결과 현 단계에서는 어려우므로, 기존 Notification 기능을 활용하여 안내한다.
- **에러 표시 원칙:** 에러 상세 정보는 hover/tooltip이 아닌 지속적으로 확인 가능한 형태여야 한다 (PM 결정). 구체적인 표현 방식은 디자이너가 결정한다.
- **진행률 산정:** 남은 시간 표기 불가 (개발팀 확인). Step 단위로 20%씩 균등 분배.
- **명칭 변경:** "Report Logs" → **"User Action Logs"** (Report Edit & Approve, Send to PACS 등 User Action만 표시)
- **Job Logs 스텝명:** "Store end" → **"Store non-review reports"** (Review가 필요하지 않는 Report들만 자동 Store 했다는 의미)
- **Design 카드:** [Figma - SwiftSight Product Redesign](https://www.figma.com/design/jIBORdxDlQozXYBk5j75zz/SwiftSight-Product-Redesign?node-id=13548-42965&t=qhCksN30IdQXWl2L-11)

---

## PM에게 확인 필요한 사항 (Use Case 작성 과정에서 도출)

| # | 질문 | 배경 | 결정 |
|---|------|------|------|
| 1 | **진행률 갱신 주기** | 실시간(WebSocket/SSE) vs 주기적 polling vs 수동 새로고침? | WebSocket 기반 실시간 (ABE-77) |
| 2 | **처리 스텝 정의** | Job을 구성하는 스텝 목록이 확정되었는가? | **확정:** Processing Start → Upload End Time → Modal Analysis Time → Report Generation Time → Store non-review reports (5단계, 각 20%) |
| 3 | **에러 재시도** | 에러 발생 시 유저가 특정 스텝부터 재시도할 수 있는가? 전체 재시도만 가능한가? | |
| 4 | **복수 에러** | 한 Job에서 여러 스텝에 에러가 발생하면 모두 표시하는가? 최초 에러만 표시하는가? | |
| 5 | **Mismatching Notification 상세 수준** | series name 외에 추가로 표시해야 할 정보가 있는가? | |
| 6 | **진행률 표시 범위** | 모든 Report에 대해 표시하는가? 처리 중인 Report에만 표시하는가? | |
| 7 | **에러 이력 보존** | 재시도 후 성공한 경우 이전 에러 이력도 보존하는가? | |
| 8 | **권한** | 에러 상세 정보 조회에 Admin/User 권한 차이가 있는가? | |
| 9 | **남은 시간 표기** | Report 처리에 대한 남은 시간을 표기할 수 있는가? | **불가** — 개발팀 검토 결과 남은 시간을 알기 어려움. 스텝 기반 퍼센트로 대체 |
| 10 | **User Action Logs 명칭** | "Report Logs" vs "User Action Logs" | **User Action Logs로 확정** — Report Edit & Approve, Send to PACS 등 User Action만 표시 |
