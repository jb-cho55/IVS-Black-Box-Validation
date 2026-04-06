# Test Environment

## 1. Overview

본 문서는 IVS 제어기 Black Box Testing 프로젝트에서 구축한 검증 환경을 정리한 문서이다.

프로젝트의 검증 환경은 단순한 테스트 실행 환경이 아니라,  
요구사항 기반 검증을 수행하기 위해 **입력 ECU 구성**, **CAN 메시지 정의**, **수동 검증 환경**, **자동화 테스트 환경**까지 포함한 구조로 설계했다.

본 환경은 IVS 제어기를 중심으로 주변 ECU를 시뮬레이션하고,  
입력 조건에 따른 IVS의 동작을 확인하며, 반복 검증이 필요한 항목은 자동화 테스트 모듈로 검증할 수 있도록 구성했다.

---

## 2. Environment Objective

검증 환경 구축의 목적은 다음과 같다.

- IVS 제어기의 요구사항을 실제 메시지 기반으로 검증할 수 있는 구조를 만든다.
- 주변 ECU 입력을 시뮬레이션하여 다양한 조건을 재현한다.
- Panel과 Trace를 활용해 수동 검증이 가능하도록 한다.
- 반복 검증 항목은 자동화 테스트 환경으로 전환해 효율적으로 수행한다.
- 정적/동적 테스트 결과를 요구사항과 연결해 분석할 수 있는 기반을 만든다.

---

## 3. Environment Configuration

본 프로젝트의 검증 환경은 다음 요소로 구성된다.

- **검증 대상 제어기**: IVS
- **주변 ECU 시뮬레이션**: BMS, ENG, STR, BRK, UDS
- **네트워크 구성 도구**: CANoe
- **메시지 정의**: CANdb
- **수동 검증 도구**: Panel, Trace
- **자동화 검증 도구**: Test Environment, Test Module, CAPL 기반 테스트 코드

---

## 4. Network Configuration

### 4.1 CANoe Network Node 구성

검증 환경에서는 IVS 외 ECU를 **CANoe network node**로 구성하였다.  
이를 통해 실제 차량 환경과 유사하게 입력 ECU에서 메시지를 송신하고, IVS가 이를 수신하여 상태를 판단하는 구조를 만들었다.

구성한 주요 ECU는 다음과 같다.

- **IVS**
- **BMS**
- **ENG**
- **STR**
- **BRK**
- **UDS**

### 4.2 Network Configuration Purpose

이와 같은 네트워크 구성의 목적은 다음과 같다.

- 각 ECU 입력을 독립적으로 제어할 수 있도록 하기 위함
- Fault 검출 조건에 필요한 입력 조합을 재현하기 위함
- UDS 요청/응답 흐름을 검증하기 위함
- 실제 제어기 검증과 유사한 구조를 만들기 위함

---

## 5. CAN Database Configuration

### 5.1 CANdb 작성

IVS 제어기에서 송수신하는 CAN 메시지에 대해 **CANdb**를 작성하였다.

CANdb에는 다음 요소가 포함된다.

- ECU별 송수신 메시지
- 각 메시지의 Signal 정의
- Signal 길이 및 값 표현 방식
- Description 및 상태값 정보

### 5.2 CANdb 구성 목적

CANdb는 단순히 메시지를 정의하는 역할을 넘어서,  
요구사항 기반 검증에서 입력과 출력의 기준이 되는 핵심 요소였다.

특히 다음과 같은 이유로 중요했다.

- 각 ECU 입력 메시지를 정확히 구성하기 위함
- Fault 검출/복구 조건에서 사용하는 Signal 값을 정의하기 위함
- UDS 요청/응답 메시지를 일관되게 검증하기 위함
- 정적 테스팅에서 Signal length, 값 표현 방식, Description 불일치 여부를 검토하기 위함

---

## 6. Manual Validation Environment

### 6.1 Panel 구성

수동 테스트를 위해 **Panel**을 구성하였다.

Panel을 통해 다음과 같은 입력 제어가 가능하도록 설계했다.

- Batt_Percent
- Batt_Chg_Sts
- Batt_Voltage
- Ignition_sts
- Engine_sts
- Steering_angle
- Brake_press
- ACCEL_press
- Vehicle_Speed
- UDS 요청 트리거

Panel은 수동으로 값을 조작하며 요구사항 조건을 빠르게 재현할 수 있도록 하는 역할을 한다.

### 6.2 Trace 구성

CAN 메시지의 송수신 흐름과 Fault 응답을 확인하기 위해 **Trace 창**을 활용했다.

Trace를 통해 확인한 주요 항목은 다음과 같다.

- ECU별 입력 메시지 송신 여부
- IVS 응답 메시지 송신 여부
- UDS 요청/응답 흐름
- Fault 상태 요청에 대한 응답 타이밍
- 특정 조건에서의 메시지 변화 시점

### 6.3 Manual Validation Purpose

수동 검증 환경은 다음과 같은 목적을 가진다.

- 요구사항 조건을 빠르게 재현하고 동작을 직접 확인
- 자동화 이전 단계에서 기본 동작 검증
- 메시지 흐름 및 응답 구조 이해
- 테스트 케이스 설계 전 동작 특성 파악
- 결함 발생 시 자동화 결과와 비교할 기준 확보

---

## 7. Automated Validation Environment

### 7.1 Test Environment 구축

반복 검증이 필요한 항목에 대해서는 **자동화 테스트를 위한 Test Environment**를 구축했다.

자동화 환경은 다음과 같은 검증에 활용했다.

- 경계값 검증
- Fault Detection / Recovery 검증
- 상태 전이 검증
- IGN Off → On 50회 전환 검증
- UDS 응답 조건 검증
- 타이밍 검증

### 7.2 Test Module 생성

자동화 검증을 위해 Test Module을 생성하고,  
각 요구사항을 테스트할 수 있도록 모듈 구조를 구성했다.

테스트 모듈의 목적은 다음과 같다.

- 동일 조건을 반복 실행 가능하게 함
- 테스트 절차를 정형화함
- 결과 비교를 쉽게 함
- 수동 검증 대비 재현성과 일관성을 높임

### 7.3 CAPL 기반 자동화

자동화 검증은 CAPL 기반 테스트 코드로 수행했다.

자동화 코드에서는 다음 요소를 다뤘다.

- 입력 조건 설정
- 메시지 송신 제어
- 응답 메시지 수신 확인
- Fault 상태 판별
- 타이밍 및 조건 충족 여부 검증

---

## 8. Testcase Reuse Strategy

본 프로젝트에서는 모든 테스트 케이스를 처음부터 개별적으로 작성하지 않고,  
**기본적인 TESTCASE를 먼저 작성한 뒤 각 페이지의 요구사양에 맞춰 함수와 파라미터를 수정하는 방식**으로 재사용 구조를 적용했다.

이 방식의 장점은 다음과 같다.

- 공통 로직 재사용 가능
- 테스트 코드 중복 감소
- 요구사항별 확장 용이
- 조건별 비교 검증이 쉬움
- 유지보수성 향상

예를 들어 다음과 같은 방식으로 재사용할 수 있다.

- 동일한 Fault 검출 로직에서 입력 Signal 값만 변경
- 동일한 테스트 함수에 경계값 파라미터만 조정
- 동일한 UDS 요청 흐름에 응답 조건만 다르게 적용

---

## 9. Environment Build Flow

검증 환경 구축은 다음 순서로 진행했다.

1. 검증 대상인 IVS와 주변 ECU를 식별
2. CANoe network node 구성
3. IVS 송수신 메시지에 대한 CANdb 작성
4. 수동 테스트용 Panel 설계
5. CAN 메시지 추적을 위한 Trace 환경 구성
6. 자동화 테스트용 Test Environment 구축
7. Test Module 생성
8. CAPL 기반 자동화 테스트 코드 작성 및 적용

---

## 10. Why This Environment Matters

본 프로젝트의 검증 환경은 단순히 테스트를 실행하기 위한 설정이 아니라,  
요구사항 기반 검증을 실질적으로 가능하게 만든 핵심 기반이었다.

특히 다음과 같은 점에서 의미가 있다.

- 실제 차량 제어기 검증과 유사한 구조를 재현
- 입력 ECU와 검증 대상 제어기를 분리된 역할로 구성
- 수동 검증과 자동화 검증을 함께 운용
- 정적/동적 테스트 결과를 동일 환경에서 연결 가능
- 반복 검증과 타이밍 검증이 가능한 구조 확보

---

## 11. Summary

본 프로젝트의 테스트 환경은 다음 요소를 중심으로 구성되었다.

- CANoe network node 기반 ECU 구성
- CANdb 기반 메시지 정의
- Panel / Trace 기반 수동 검증 환경
- Test Environment / Test Module 기반 자동화 검증 환경
- CAPL 기반 반복 테스트 구조

이 환경을 기반으로 IVS 제어기의 CAN/UDS 요구사항과 Fault 관리 로직을 정적/동적 관점에서 검증할 수 있었다.
