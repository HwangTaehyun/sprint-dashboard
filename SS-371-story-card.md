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

### S1. Report별 처리 상태를 Loading Bar로 표시한다. (N1 해결)

- 각 Report의 처리 상태를 유저가 확인할 수 있어야 한다.
- ~~진행률 퍼센트 (25%, 50% 등) 표시~~ → **Loading Bar (indeterminate)** 로 변경
  - **변경 배경:** 하나의 Job(Workflow) 안에서 여러 Report 처리가 모두 병렬로 이루어지면서, ~~SM(Study Manager)처럼~~ 공통 스텝이라는 개념을 만들기가 어려워졌다. 따라서 Step별 Progress 퍼센트를 정확히 산정하는 것이 불가능하며, 각 처리 작업별 로그를 남기는 식으로 진행할 예정이다.
  - 처리 중: Loading Bar 애니메이션 표시
  - 완료: 완료 상태로 전환
  - 실패: 에러 상태로 전환

### S2. Job Detail에서 Job Logs와 User Action Logs를 표시한다. (N2 해결)

- **Job Logs:** 시간+텍스트 형식의 상세 처리 로그를 표시한다.
  - 하나의 Job(Workflow) 안에서 여러 Report가 병렬로 처리되므로, ~~SM처럼~~ 공통 Step 기반의 구조가 아닌 **각 처리 작업별 시간순 이벤트 로그** 형식으로 표시한다.
  - 각 로그 라인: `[타임스탬프] [상태 아이콘] [이벤트 메시지]`
  - 상태 아이콘: ✅ 성공, ❌ 에러, ⏳ 처리 중
  - 로그가 길 수 있으므로 **Scroll 가능**해야 함
  - 주요 이벤트 유형:
    - Workflow started / completed / failed
    - Registered patient, study, series, reports
    - Uploaded series (series name + instance count)
    - Model analysis completed / failed / in progress (model name)
    - Report generated (report name)
    - Sent to PACS / PACS transmission failed (output name)
  - 에러 발생 시: 에러 메시지를 로그 라인 다음 줄에 들여쓰기로 상세 표시
  - Workflow 완료/실패 시: 총 소요 시간 표시
- **User Action Logs:** (기존 "Report Logs"에서 명칭 변경) Report에 대한 사용자 액션 이력을 보여준다.
  - 표시 대상 액션: Report Edit, Report Approve, Send to PACS 등의 User Action들

#### Job Log 예시

**Case 1: 정상 완료**
```
2026-04-01 10:00:00   ✅  Workflow started
2026-04-01 10:00:01   ✅  Registered patient, study, series, reports
2026-04-01 10:00:02   ✅  Uploaded series: brain_3d_t1 (124 instances)
2026-04-01 10:00:08   ✅  Uploaded series: brain_t2_flair (30 instances)
2026-04-01 10:01:45   ✅  Model analysis completed: Volumetry
2026-04-01 10:02:30   ✅  Model analysis completed: WMH 2D
2026-04-01 10:03:05   ✅  Report generated: Brain Dementia Report (Physician)
2026-04-01 10:03:06   ✅  Sent to PACS: Volumetry output (segmented DICOM)
2026-04-01 10:03:07   ✅  Sent to PACS: WMH 2D output
2026-04-01 10:03:08   ✅  Sent to PACS: Brain Dementia Report (Physician)
2026-04-01 10:03:08   ✅  Workflow completed (3m 8s)
```

**Case 2: Model 실패 — Timeout**
```
2026-04-01 14:30:00   ✅  Workflow started
2026-04-01 14:30:01   ✅  Registered patient, study, series, reports
2026-04-01 14:30:02   ✅  Uploaded series: brain_3d_t1 (124 instances)
2026-04-01 14:30:06   ✅  Uploaded series: brain_t2_flair (30 instances)
2026-04-01 14:31:50   ✅  Model analysis completed: Volumetry
2026-04-01 14:40:06   ❌  Model analysis failed: WMH 2D
                          Error: StartToCloseTimeout exceeded (10m0s)
2026-04-01 14:40:06   ❌  Workflow failed (10m 6s)
```

**Case 3: PACS 전송 실패**
```
2026-04-01 16:00:00   ✅  Workflow started
...
2026-04-01 16:02:15   ✅  Report generated: Brain Atrophy Report (Physician)
2026-04-01 16:02:16   ❌  PACS transmission failed: Volumetry output
                          Error: failed to download from S3: connection timeout
2026-04-01 16:02:16   ❌  Workflow failed (2m 16s)
```

**Case 4: 처리 중**
```
2026-04-01 17:00:00   ✅  Workflow started
...
2026-04-01 17:01:45   ✅  Model analysis completed: Volumetry
2026-04-01 17:02:00   ⏳  Model analysis in progress: WMH 3D
                     ...  waiting
```

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
| FR-1 | Report별 처리 상태를 Loading Bar (indeterminate)로 표시한다. ~~Step별 퍼센트 표시 제거~~ | S1 | 중상 |
| FR-2 | Job Detail에서 Job Logs — 시간순 이벤트 로그를 표시한다 (타임스탬프 + 상태 아이콘 + 메시지) | S2 | 상 |
| FR-2-1 | Job Logs 영역은 Scroll 가능해야 한다 (로그가 길 수 있음) | S2 | 상 |
| FR-2-2 | Job Detail에서 User Action Logs — Report Edit, Approve, Send to PACS 등 사용자 액션 이력을 표시한다 | S2 | 상 |
| FR-3 | 에러 발생 이벤트를 ❌ 아이콘으로 식별하고, 에러 메시지를 다음 줄에 상세 표시한다 | S3 | 상 |
| FR-4 | 에러 메시지를 상시 노출한다 (hover 방식 아님) | S3 | 상 |
| FR-5 | PACS 전송 에러 메시지를 표시한다 | S3 | 상 |
| FR-6 | Upload/Download timeout 에러를 표시한다 | S3 | 상 |
| FR-7 | Mismatching 발생 시 series name을 포함한 Notification을 생성한다 | S4 | 상 |
| FR-8 | 상태 아이콘 3종 필요: ✅ 성공, ❌ 에러, ⏳ 처리 중 | S2, S3 | 상 |
| FR-9 | Workflow 완료/실패 시 총 소요 시간을 표시한다 | S2 | 중 |

---

## 참고 사항

- **Report 병렬 처리 (핵심 변경 사유):** 설계 과정에서 하나의 Job(Workflow) 안에서 여러 Report 처리가 모두 병렬로 이루어지는 구조가 확정됨. 이에 따라 ~~SM(Study Manager)처럼~~ 공통 스텝이라는 개념을 만들기가 어려워졌고, 공통 Step 기반 퍼센트(20%씩 5단계) 구조가 불가능해짐. 대신 **각 처리 작업별 로그를 남기는 식**으로 진행하며, 시간순 이벤트 로그 형식으로 변경.
- **Progress 표시 변경:** ~~Step별 퍼센트 (20%, 40%, 60%...)~~ → **Loading Bar (indeterminate)** 로 변경. 병렬 처리 구조에서 정확한 퍼센트 계산이 어려움.
- **Job Logs Scroll:** 로그가 길 수 있으므로 Scroll 가능한 영역으로 구현 필요.
- **상태 아이콘 3종:** ✅ 성공, ❌ 에러/실패, ⏳ 처리 중 — 디자이너 아이콘 제작 필요.
- **에러 표시 원칙:** 에러 상세 정보는 hover/tooltip이 아닌 지속적으로 확인 가능한 형태. 로그 라인 다음 줄에 들여쓰기로 에러 메시지 표시.
- **명칭 변경:** "Report Logs" → **"User Action Logs"**
- **Mismatching UI:** 기존 Notification 기능 활용
- **Design 카드:** [Figma - SwiftSight Product Redesign](https://www.figma.com/design/jIBORdxDlQozXYBk5j75zz/SwiftSight-Product-Redesign?node-id=13548-42965&t=qhCksN30IdQXWl2L-11)
- **변경 이력:**
  - v1: Step 기반 퍼센트 (5단계, 각 20%) — 초기 기획
  - v2 (현재): 시간순 이벤트 로그 + Loading Bar — Report 병렬 처리 구조 반영

---

## PM에게 확인 필요한 사항 (Use Case 작성 과정에서 도출)

| # | 질문 | 배경 | 결정 |
|---|------|------|------|
| 1 | **진행률 갱신 주기** | 실시간(WebSocket/SSE) vs 주기적 polling? | WebSocket 기반 실시간 (ABE-77) |
| 2 | **처리 스텝 정의** | Job을 구성하는 스텝 목록이 확정되었는가? | **v2 변경:** ~~공통 5단계 Step~~ → 시간순 이벤트 로그 (Report 병렬 처리로 공통 Step 불가) |
| 3 | **에러 재시도** | 에러 발생 시 특정 스텝부터 재시도 가능? | |
| 4 | **복수 에러** | 한 Job에서 여러 이벤트 에러 시 모두 표시? | **확정:** 모든 에러를 시간순으로 로그에 표시 (병렬 처리이므로 여러 에러 가능) |
| 5 | **Mismatching Notification 상세 수준** | series name 외 추가 정보? | |
| 6 | **진행률 표시 방식** | Report별 진행률을 어떻게 표시? | **확정:** Loading Bar (indeterminate). ~~Step별 퍼센트~~ 불가 (병렬 처리) |
| 7 | **에러 이력 보존** | Retry 후 이전 에러도 보존? | |
| 8 | **권한** | Admin/User 에러 조회 권한 차이? | |
| 9 | **남은 시간 표기** | 남은 시간 표기 가능? | **불가** — Loading Bar로 대체 |
| 10 | **User Action Logs 명칭** | "Report Logs" vs "User Action Logs" | **User Action Logs로 확정** |
| 11 | **Job Logs Scroll** | 로그가 길 때 UI 처리 방식? | **확정:** Scroll 가능한 영역으로 구현 |
| 12 | **상태 아이콘** | 어떤 아이콘이 필요? | **확정:** ✅ 성공, ❌ 에러, ⏳ 처리 중 (디자이너 제작 필요) |
