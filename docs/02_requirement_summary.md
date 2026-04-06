# Requirement Summary

## 1. Overview

본 문서는 IVS 제어기를 대상으로 한 Black Box Testing 프로젝트의 요구사항을 테스트 관점에서 재정리한 문서이다.  
검증 범위는 CAN 기반 입력 메시지 해석, UDS 요청/응답 처리, Fault 검출/복구/삭제, 상태 전이, 그리고 경계값 및 타이밍 조건을 포함한다.

---

## 2. System Context

IVS 제어기는 주변 ECU로부터 입력 메시지를 수신하고,  
요구사항에 따라 Fault 상태를 판단하며, UDS 요청에 대한 응답 메시지를 송신해야 한다.

### Related ECU

- **BMS**
- **ENG**
- **STR**
- **BRK**
- **UDS**

---

## 3. CAN Input Requirements

IVS는 아래 입력 신호들을 해석해야 한다.

| ECU | Signal | Description |
|---|---|---|
| BMS | Batt_Percent | 배터리 충전량 |
| BMS | Batt_Chg_Sts | 배터리 충전 상태 |
| BMS | Batt_Voltage | 배터리 전압 |
| ENG | Ignition_sts | 시동 상태 |
| ENG | Engine_sts | 엔진 상태 |
| STR | Steering_angle | 조향각 |
| BRK | Brake_press | 브레이크 압력 |
| BRK | ACCEL_press | 악셀 압력 |
| BRK | Vehicle_Speed | 차량 속도 |

### Input Requirement Summary

- IVS는 각 ECU에서 전달되는 입력 메시지를 요구사항에 맞게 해석해야 한다.
- 입력 조합에 따라 Fault Detection / Recovery / Delete 로직이 달라진다.
- 일부 Fault는 단일 입력만이 아니라 여러 입력의 조합 조건을 만족해야 검출된다.
- 경계값 조건(`15%`, `80%`, 전압 기준, 조향각 기준 등)은 구현과 요구사항의 일치 여부를 반드시 검증해야 한다.

---

## 4. UDS Request / Response Requirements

IVS는 UDS 계열 요청 메시지를 수신하고 적절한 응답을 송신해야 한다.

| Request Type | Expected Behavior |
|---|---|
| Software Version Request | 소프트웨어 버전 정보를 포함한 응답 송신 |
| Fault Status Request | 현재 유효한 고장 상태를 포함한 응답 송신 |
| Fault Delete Request | 전체 고장 삭제 요청 처리 및 결과 반영 |

### Additional UDS Requirement

- Fault 상태 요청에 대한 응답에는 **현재 유효한 고장 상태**가 포함되어야 한다.
- 응답 후 **10ms 이내**에 들어온 동일 고장 상태 요청은 무시해야 한다.

---

## 5. Fault Management Requirements

고장 관리 로직은 본 프로젝트의 핵심 검증 대상이다.

### 5.1 Fault Detection

- 모든 고장은 **10ms 주기로 동작하는 SW**에서 연속 감지될 때 성립해야 한다.
- 검출 조건이 만족되면 즉시 고장 확정이 이루어져야 한다.
- 일부 Fault는 Pre-condition과 Detection condition을 모두 만족해야 한다.

### 5.2 Fault Recovery

- 고장 복구는 **이미 고장이 확정된 경우에만 가능**하다.
- Recovery 조건은 Detection 조건과 별도로 검증해야 한다.
- 전압, 상태값, 조건 유지 시간 등 Recovery 기준은 요구사항과 정확히 일치해야 한다.

### 5.3 Fault State

Fault 상태는 아래와 같이 관리된다.

| State | Value | Meaning |
|---|---:|---|
| Deleted | 0 | 고장이 삭제된 상태 |
| Fixed | 1 | 고장이 복구된 상태 |
| Faulted | 2 | 고장이 검출된 상태 |

### 5.4 Fault Delete Condition

- **IGN Off → On 전환이 50회 발생하면** 고장을 삭제할 수 있어야 한다.
- 해당 조건은 단순 카운트 확인이 아니라 실제 동작 시점까지 검증해야 한다.

---

## 6. Timing Requirements

본 프로젝트에서 타이밍은 중요한 검증 포인트다.

### Main Timing Points

- 모든 고장은 **10ms 주기 SW** 기준으로 동작
- 동일 Fault 상태 요청은 **응답 후 10ms 이내 재요청 시 무시**
- 일부 Fault는 특정 시간 이내 또는 특정 시간 유지 후 검출/미검출이 결정됨
- Fault 검출 여부뿐 아니라 **검출 시점의 정확성**도 검증 대상

---

## 7. Boundary Conditions to Validate

요구사항 기반 테스트에서 특히 중요하게 확인해야 하는 경계값 조건은 다음과 같다.

- `Batt Percent = 15%`
- `Batt Percent = 80%`
- `IGN Off → On = 50 cycle`
- 배터리 전압 기준값
- 조향각 기준값
- Ignition / Engine 상태 조합 조건

이러한 경계값은 구현 시 `<`, `<=`, `>`, `>=` 처리 차이로 인해 실제 동작이 달라질 수 있으므로 반드시 별도 검증이 필요하다.

---

## 8. Requirement Interpretation Points

요구사항을 테스트 관점에서 해석할 때 중요했던 포인트는 다음과 같다.

### 8.1 Signal Definition Consistency
- Signal length가 상태 수를 충분히 표현할 수 있는지 확인해야 한다.
- Signal name과 Description이 서로 일치하는지 확인해야 한다.
- 값 표현 방식(10진수 / 16진수)이 요구사항과 일치하는지 확인해야 한다.

### 8.2 Pre-condition Validation
- Fault가 특정 조건에서만 검출되어야 하는 경우,
  Detection condition뿐 아니라 Pre-condition 만족 여부도 함께 검증해야 한다.

### 8.3 Boundary Validation
- 15%, 80%, 50 cycle과 같은 경계값은 단일 포인트가 아니라
  직전 값 / 해당 값 / 초과 값을 함께 비교해야 한다.

### 8.4 Recovery Validation
- Fault 검출만 확인하는 것이 아니라,
  복구 조건에서 상태가 올바르게 전이되는지 함께 확인해야 한다.

---

## 9. Test Design Implications

요구사항 분석 결과, 테스트 케이스는 다음 방향으로 설계할 필요가 있었다.

- 입력 ECU별 기본 테스트 케이스 작성
- Fault별 Pre-condition / Detection / Recovery / Delete 조건 분리 검증
- 경계값 중심의 테스트 데이터 설계
- 타이밍 포함 검증 로직 설계
- 기본 테스트 케이스 작성 후 함수 및 파라미터를 수정하는 방식으로 재사용

---

## 10. Summary

본 프로젝트의 요구사항은 단순 입력-출력 확인 수준이 아니라,  
다양한 ECU 입력 조합, UDS 요청/응답, Fault 상태 전이, 경계값, 타이밍 조건까지 포함하는 상태 기반 검증 구조로 구성되어 있다.

따라서 테스트 설계 시 다음 요소를 함께 고려해야 한다.

- 입력 신호 해석 정확성
- UDS 응답 조건
- Fault Detection / Recovery / Delete
- Deleted / Fixed / Faulted 상태 전이
- 경계값 처리
- 타이밍 조건
- Pre-condition 반영 여부
