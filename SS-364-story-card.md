# SS-364: SwiftCL Sequential 기능 업데이트 — Activity 병렬처리

---

## User Needs

### N1: Job 내 Activity 묶음별 병렬처리로 처리 속도 개선 (우선순위: 상)

**구체적 문제점:** 기존에는 Job 내에서 각 모델 처리 및 리포트 생성이 순차적으로 실행되어, 전체 처리 시간이 개별 Activity의 합산이었다. 특히 Volumetry Model 처리가 오래 걸리는 경우, 뒤에 대기하는 WMH Model 및 Report 생성이 불필요하게 지연되었다.

**기대 효과:** 
- 각 모델 및 리포트가 만들어져야 하는 Activity 묶음별로 병렬처리되어 전체 처리 시간 단축
- Volumetry Report 4종 + WMH Report 1종이 선택되더라도 거의 동시에 완료

---

## Solutions

**해결책 유형:** ✅ Improvement

### S1. Job 내 Activity 묶음별 병렬처리

- 3D_T1 시리즈 → Volumetry Model 처리
- 2D_FLAIR 시리즈 → WMH 2D Model 처리
- 위 두 경로를 **병렬로 동시 실행**
- Volumetry Model 처리가 WMH 2D Model 처리보다 오래 걸리지만, Report 생성은 매우 빠르므로 최종 결과물은 거의 동시에 완료됨

### S2. Model Output → PACS 전송 우선순위

- Model 처리가 끝난 output들이 Report보다 **먼저 PACS에 전송**되도록 순서 보장
- Report 생성 완료 후 Report도 PACS에 전송

### S3. Store 로직 변경

- 병렬처리에 맞춰 Store 관련 코드도 수정됨

---

## 기능 요구사항

| ID | 요구사항 | 관련 Solution | 우선순위 |
|----|---------|--------------|---------|
| FR-1 | Job 내 Activity 묶음별 병렬처리 지원 | S1 | 상 |
| FR-2 | Model output이 Report보다 먼저 PACS 전송 | S2 | 상 |
| FR-3 | Store 로직 병렬처리 대응 | S3 | 중 |

---

## 테스트 포인트

이 변경에 대해 특별히 확인해야 할 항목:

### TP-1: Model Output PACS 전송 순서

- Model 처리가 끝난 output들이 Report보다 **먼저** PACS에 전송되는지 확인
- Volumetry Model output → PACS 전송 완료 → Report 생성 → Report PACS 전송 순서 검증

### TP-2: 병렬처리 정합성

- Volumetry Report 4종 + WMH Report 1종 선택 시, 거의 동시에 생성되는지 확인
- 각 Report의 내용이 정확한지 (병렬 처리로 인한 데이터 혼선 없는지)

### TP-3: Store 변경 영향

- Store 관련 코드 변경에 따른 기존 기능 정상 동작 확인
- Store non-review reports 동작 검증

---

## 참고 사항

- **처리 시간 참고:**
  - 3D_T1 → Volumetry Model: 상대적으로 오래 걸림
  - 2D_FLAIR → WMH 2D Model: Volumetry보다 빠름
  - Report 생성: 매우 빠름
  - 결과적으로 Volumetry 4종 + WMH 1종 = 거의 동시 완료
- **기존 순차 처리 대비 변경:** 순차 → 병렬, 전체 처리 시간 = max(개별 Activity 시간)으로 단축
- **관련 BE 카드:** ABE-87 (Workflow Retry From Scratch), ABE-80/81 (V3 Pipeline, DSL Compiler)

---

## 확인 필요한 사항

| # | 질문 | 결정 |
|---|------|------|
| Q1 | 병렬처리 실패 시 롤백 범위 | 실패한 Activity만 재시도? 전체 Job 재시도? |
| Q2 | PACS 전송 순서 보장 방식 | Model output 전송 완료 대기 후 Report 전송? 또는 순차 큐? |
| Q3 | Store 변경에 따른 하위 호환성 | 기존 순차 처리 Job과의 호환성 확인 필요 |
