# Phase 1: 스키마/용어사전 정의서 — 통합 경정청구 환급액 산출 시스템

> **Project**: TaxServiceENTEC-ATR
> **Version**: v1.0
> **Date**: 2026-02-18
> **Status**: Draft
> **참조**: refer-to-doc/참고1번-development-design-v2.md (방향성 참조), refer-to-doc/법인세·종합소득세-프롬프트_v2.0.md
> **설계 원칙**: 참고1번 설계서는 초안으로 방향성만 참조. 입력·요약·출력 3계층을 새롭게 정의.

---

## 목차

1. [용어사전 (Glossary)](#1-용어사전-glossary)
2. [도메인 엔티티 정의](#2-도메인-엔티티-정의)
3. [데이터 아키텍처 — 3계층 + 기준정보 + 감사](#3-데이터-아키텍처)
4. [Layer 1: 입력자료 보관 테이블 (INP_)](#4-layer-1-입력자료-보관-테이블)
5. [Layer 2: 중간 요약 테이블 (SMR_)](#5-layer-2-중간-요약-테이블)
6. [Layer 3: 산출 결과 / 출력 보관 테이블 (OUT_)](#6-layer-3-산출-결과-출력-보관-테이블)
7. [Layer 4: 기준정보 테이블 (REF_)](#7-layer-4-기준정보-테이블)
8. [Layer 5: 감사 로그 테이블 (AUD_)](#8-layer-5-감사-로그-테이블)
9. [JSON 데이터셋 구조 — 입력/출력 스펙](#9-json-데이터셋-구조)
10. [엔티티 관계도](#10-엔티티-관계도)
11. [테이블 요약 매트릭스](#11-테이블-요약-매트릭스)

---

## 1. 용어사전 (Glossary)

### 1.1 비즈니스 용어 (세무 도메인)

| 한글 용어 | 영문 (코드) | 정의 | 매핑 |
|----------|-----------|------|------|
| 경정청구 | AmendmentClaim | 기존 세금 신고에 오류가 있을 때 과다 납부 세액의 환급을 요청하는 절차 (국기법 §45의2) | refund_request |
| 법인세 | CORP (Corporate Tax) | 법인의 각 사업연도 소득에 부과되는 세금 (법인세법) | tax_type = 'CORP' |
| 종합소득세 | INC (Income Tax) | 개인의 종합소득에 부과되는 세금 (소득세법) | tax_type = 'INC' |
| 세액공제 | TaxCredit | 산출세액에서 직접 차감하는 금액 (§24 투자, §29의8 고용, §10 R&D 등) | credit |
| 세액감면 | TaxExemption | 산출세액의 일정 비율을 면제하는 금액 (§6 창업, §7 중소특별) | exemption |
| 과세표준 | TaxBase | 세율을 적용하기 전의 세금 계산 기준 금액 | tax_base |
| 산출세액 | CalculatedTax | 과세표준 × 세율로 산출한 세금 | calculated_tax |
| 결정세액 | DeterminedTax | 산출세액에서 공제/감면 적용 후 확정된 세금 | determined_tax |
| 최저한세 | MinimumTax | 감면/공제 후에도 최소로 납부해야 하는 세액 (조특법 §132) | minimum_tax |
| 상호배제 | MutualExclusion | 동시에 적용할 수 없는 공제/감면 조합 규칙 (조특법 §127④) | exclusion_rule |
| 농어촌특별세 | AgriculturalTax | 일부 공제/감면에 대해 20% 부과되는 별도 세금 | agricultural_tax |
| 환급가산금 | RefundInterest | 환급 시 경과일수에 대해 지급하는 이자 상당액 | refund_interest |
| 상시근로자 | RegularEmployee | 고용보험 피보험자 중 제외대상을 뺀 연평균 인원 수 | regular_employee_count |
| 이월공제 | CarryforwardCredit | 당해 미사용 공제액을 향후 연도에 이월 적용 (최대 10년) | carryforward |
| 이월결손금 | LossCarryforward | 과거 사업연도 결손금을 이월하여 소득에서 공제 (최대 15년) | loss_carryforward |
| 소급공제 | LossCarryback | 당해 결손금으로 직전 연도 납부세액 환급 (중소기업 한정) | loss_carryback |
| 감면소득배분 | ExemptIncomeAllocation | 산출세액 × (감면대상소득 / 총소득) × 감면율 | income_allocation |
| 점검항목 | InspectionItem | 환급 가능성을 진단하는 개별 검토 항목 (법인 40개 + 개인 37개 = 77개) | inspection_item |
| 기준정보 | ReferenceData | req_id 무관하게 독립 관리되는 세율/공제율/한도 등 공통 기준 | reference_data |
| 절사 | Truncation | 금액 10원 미만, 비율 소수점 3자리를 버리는 정책 (반올림 절대 금지) | truncation |

### 1.2 세목별 핵심 차이 용어

| 개념 | 법인세 (CORP) | 종합소득세 (INC) |
|------|-------------|----------------|
| 소재지 판단 | 본점 간주: 본점=수도권이면 전 사업장 수도권 (§7 단서) | 사업장별 개별 판단 (본점 간주 없음) |
| 최저한세 기준 | 과세표준 × 최저한세율 (7~17%) | 산출세액 × 35%/45% |
| 적용순서 법령 | 법인세법 §59 | 소득세법 §60 |
| 결손금 이월 | 법인세법 §13, 중소 100% / 일반 80% | 소득세법 §45, 중소 100% |
| 결손금 소급 | 법인세법 §72 (중소기업) | 소득세법 §85의2 (중소기업) |
| 감면세액 분모 | 각 사업연도 소득 | 종합소득금액 |
| 환급가산금 기산 | 납부일 다음날 (중간예납 별도) | 법정납부기한 다음날 (5.31 / 성실 6.30) |
| 환급가산금 세무처리 | 익금불산입 (법인세법 §18④) | 비과세 소득 (열거주의) |
| 근로자 제외 대상 | 임원, 최대주주 4촌 이내 | 대표자 본인, 배우자, 직계존비속 |
| 다사업장 | 법인 전체 합산 (소재지 예외: 투자는 실제소재지) | 사업장별 각각 판단 |
| 전용 공제/감면 | 외국납부(법인§57), 수입배당(§18의2), 세무조정, 감가상각 | 소득공제 최적화, 성실사업자, 기장세액, 노란우산, 연금저축, 착한임대인 |

### 1.3 글로벌 표준 / 기술 용어

| 용어 | 정의 | 참조 |
|------|------|------|
| req_id | 경정청구 요청의 유일 식별자. 형식: `{C/I}-{사업자번호10}-{날짜8}-{일련번호3}` | 시스템 자체 정의 |
| KSIC | 한국표준산업분류 코드 (통계청) | Korean Standard Industrial Classification |
| FIFO | 선입선출. 이월결손금 공제 순서에 적용 | First In, First Out |
| Branch & Bound | 조합 최적화 알고리즘 (항목 ≤15개 시 적용) | Operations Research |
| Greedy | 탐욕 알고리즘 (항목 >15개 시 폴백) | Operations Research |
| Idempotency-Key | API 중복 요청 방지용 고유 키 (UUID) | RFC 7232 |
| AES-256 | 주민번호/사업자번호 등 개인정보 암호화 표준 | Advanced Encryption Standard |
| JWT | JSON Web Token. req_id 토큰화 및 RBAC 인증 | RFC 7519 |

### 1.4 용어 사용 규칙

1. **코드에서**: 영문 사용 (`taxType`, `calculatedTax`, `regularEmployeeCount`)
2. **DB 컬럼에서**: snake_case (`tax_type`, `calculated_tax`, `regular_employee_count`)
3. **API 응답에서**: camelCase (`taxType`, `calculatedTax`)
4. **UI/문서에서**: 한글 사용 (법인세, 산출세액, 상시근로자 수)
5. **세목 코드**: `CORP` (법인), `INC` (개인) — 절대 `C`/`I` 약어 단독 사용 금지 (req_id 내부에서만 C/I)

---

## 2. 도메인 엔티티 정의

### 2.1 핵심 엔티티

| 엔티티 | 영문 | 설명 | 식별자 |
|--------|------|------|--------|
| **경정청구 요청** | Request | 1건의 경정청구 진단 작업 단위 | req_id |
| **납세자** | Taxpayer | 법인 또는 개인사업자 | taxpayer_id (사업자번호) |
| **신청인** | Applicant | 경정청구를 요청한 세무사/법인담당자 | applicant_id |
| **입력 데이터셋** | InputDataset | JSON 형태로 수신한 세부 입력자료 | req_id + category |
| **입력 요약** | InputSummary | 세부 입력에서 파생한 계산용 중간 요약 | req_id + summary_type |
| **점검항목** | InspectionItem | 77개 개별 공제/감면 점검 대상 | item_code (M4-01 등) |
| **산출결과** | CreditResult | 개별 공제/감면 계산 결과 | req_id + provision_code |
| **최적조합** | OptimalCombination | 상호배제 적용 후 환급 극대화 조합 | req_id + combo_rank |
| **환급결과** | RefundResult | 최종 환급액 및 환급가산금 | req_id |
| **보고서** | Report | JSON 8섹션 직렬화 보고서 | req_id |
| **기준정보** | ReferenceData | 세율/공제율/한도 등 독립 기준 데이터 | ref_code + effective_year |
| **감사 로그** | AuditLog | 모든 계산 이력 및 변경 추적 | log_id |

### 2.2 엔티티 관계

```
                          1건의 경정청구
                               │
                    ┌──────────┴──────────┐
                    │    Request (요청)     │
                    │    PK: req_id        │
                    │    FK: applicant_id  │
                    │    FK: taxpayer_id   │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
     ┌────────▼────────┐ ┌────▼─────┐ ┌────────▼────────┐
     │  INPUT LAYER     │ │  SUMMARY │ │  OUTPUT LAYER    │
     │  (입력 보관)     │ │  LAYER   │ │  (결과 보관)     │
     │                  │ │  (중간    │ │                  │
     │ INP_RAW_DATASET  │ │  요약)   │ │ OUT_CREDIT_DETAIL│
     │ INP_CORP_*       │ │          │ │ OUT_COMBINATION  │
     │ INP_INC_*        │ │ SMR_*    │ │ OUT_REFUND       │
     │ INP_COMMON_*     │ │          │ │ OUT_REPORT_JSON  │
     └──────────────────┘ └──────────┘ └──────────────────┘
              │                │                │
              │       ┌───────┘                │
              ▼       ▼                        │
     ┌────────────────────┐                    │
     │  REFERENCE LAYER   │◄───────────────────┘
     │  (기준정보)         │    (참조)
     │  REF_* 테이블       │
     └────────────────────┘
              │
     ┌────────▼────────┐
     │  AUDIT LAYER     │
     │  (감사 로그)     │
     │  AUD_*           │
     └──────────────────┘
```

---

## 3. 데이터 아키텍처

### 3.1 설계 원칙

| 원칙 | 설명 | 근거 |
|------|------|------|
| **3계층 분리** | 입력(INP) / 요약(SMR) / 출력(OUT)을 물리적으로 분리 | 사용자 요구: "세부 입력자료와 나누어 설계" |
| **JSON-In / JSON-Out** | 입력은 JSON 데이터셋으로 수신·보관, 출력은 JSON 변환용 정규 테이블 | 사용자 요구: "json형태의 데이터 셋으로 받아서 보관" |
| **원본 불변성** | INP_RAW_DATASET은 INSERT ONLY, 수정 시 새 req_id 발급 | 참고1번 방향성 유지 |
| **요약 재생성** | SMR_* = INP_RAW_DATASET에서 언제든 재파싱 가능 (파생 데이터) | 참고1번 방향성 유지 |
| **기준정보 독립** | REF_* = req_id 없이 독립 관리, 연도별 버전 | 참고1번 방향성 유지 |
| **세목 분기** | tax_type(CORP/INC)으로 활성화 테이블/모듈 자동 결정 | 참고1번 방향성 유지 |
| **절사 원칙** | 금액 10원 미만 TRUNCATE, 비율 소수점 3자리, 반올림 절대 금지 | 국세기본법 §86 |
| **req_id 관통** | 1건 경정청구 = 1 req_id가 전 계층을 관통하는 유일 키 | 핵심 설계 |

### 3.2 테이블 접두어 규칙

| 접두어 | 계층 | 용도 | req_id 의존 |
|--------|------|------|:----------:|
| `INP_` | Layer 1: 입력 | JSON 데이터셋 원본 보관 + 파싱된 상세 | O |
| `SMR_` | Layer 2: 요약 | 기준정보 활용 계산용 중간 요약 | O |
| `OUT_` | Layer 3: 출력 | 산출 결과, JSON 직렬화 대상 | O |
| `REF_` | Layer 4: 기준정보 | 세율/공제율/한도 등 독립 기준 | X |
| `AUD_` | Layer 5: 감사 | 계산 이력, 검증 로그 | O |

### 3.3 세목별 테이블 접미어 규칙

| 접미어 | 적용 대상 | 예시 |
|--------|----------|------|
| `_C_` | 법인 전용 | INP_C_REPRESENTATIVE, REF_C_CORP_TAX_RATE |
| `_I_` | 개인 전용 | INP_I_BUSINESS, REF_I_INCOME_TAX_RATE |
| `_S_` | 공통 (Shared) | INP_S_EMPLOYEE_DETAIL, REF_S_MUTUAL_EXCLUSION |
| 없음 | 세목 무관 | INP_RAW_DATASET, OUT_REFUND |

### 3.4 데이터 흐름

```
[외부] JSON 데이터셋 수신
   │
   ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1: 입력자료 보관 (INP_)                                    │
│                                                                 │
│  INP_REQUEST ◄── 요청 마스터 (req_id 발급)                       │
│       │                                                         │
│  INP_RAW_DATASET ◄── JSON 원본 보관 (category별, INSERT ONLY)   │
│       │ 파싱                                                    │
│  INP_S_EMPLOYEE_DETAIL ◄── 근로자 상세 (파싱된 정규 데이터)      │
│  INP_S_INVESTMENT ◄── 투자자산 상세                              │
│  INP_C_* / INP_I_* ◄── 세목별 상세 입력                         │
└────────────────────────────┬────────────────────────────────────┘
                             │ 기준정보(REF_*) 활용하여 요약 생성
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 2: 중간 요약 (SMR_)                                        │
│                                                                 │
│  SMR_BASIC_INFO ◄── 납세자 기초정보 요약 (규모/업종/소재지)       │
│  SMR_EMPLOYEE ◄── 고용 현황 요약 (상시근로자/청년등/증감)         │
│  SMR_DEDUCTION_ITEM ◄── 공제/감면 항목별 요약 (적격/금액/한도)    │
│  SMR_FINANCIAL ◄── 재무/세무 수치 요약 (과세표준/세액/결손금)      │
│  SMR_ELIGIBILITY ◄── 적격성 판정 결과 (사전점검 통과/불가)        │
│  SMR_PREP ◄── 사전준비 요약 (적용법령/후보조항/기준정보스냅샷)     │
│  SMR_VALIDATION_LOG ◄── 검증 결과 로그                           │
└────────────────────────────┬────────────────────────────────────┘
                             │ 계산 엔진 (M4~M6) 실행
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ Layer 3: 산출 결과 / 출력 보관 (OUT_)                             │
│                                                                 │
│  OUT_CREDIT_DETAIL ◄── 25개 서브모듈 개별 산출 결과               │
│  OUT_COMBINATION ◄── 최적 조합 후보 (1~N 순위)                   │
│  OUT_EXCLUSION_VERIFY ◄── 상호배제 적용 내역                     │
│  OUT_REFUND ◄── 환급액/추징/가산금 최종 결과                     │
│  OUT_RISK ◄── 사후관리 리스크 평가                               │
│  OUT_REPORT_JSON ◄── 8섹션 JSON 보고서 직렬화                    │
│  OUT_I_LOSS_CARRYBACK ◄── 개인 소급공제 결과 (INC 전용)          │
│  OUT_I_INCOME_DEDUCTION ◄── 개인 소득공제 최적화 결과 (INC 전용)  │
│  OUT_C_TAX_ADJUSTMENT ◄── 법인 세무조정 결과 (CORP 전용)         │
└─────────────────────────────────────────────────────────────────┘
         │
         ▼ JSON 직렬화
    [외부] 8섹션 JSON 보고서 응답
```

---

## 4. Layer 1: 입력자료 보관 테이블

> **목적**: 외부에서 JSON 형태의 데이터셋으로 수신한 세부 입력자료를 원본 그대로 보관하고, 계산에 필요한 정규 데이터로 파싱하여 보관한다.

### 4.1 INP_REQUEST — 요청 마스터

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | 요청 식별자 `{C/I}-{사업자번호10}-{날짜8}-{일련번호3}` |
| applicant_id | VARCHAR(20) | | O | 신청인 ID (세무사/법인) |
| taxpayer_id | VARCHAR(20) | | O | 납세자 사업자등록번호 |
| tax_type | VARCHAR(4) | | O | 세목: `CORP` / `INC` |
| tax_year | SMALLINT | | O | 경정청구 대상 과세연도 (예: 2024) |
| request_status | VARCHAR(20) | | O | 상태: received → parsing → parsed → preparing → checking → calculating → optimizing → reporting → completed / error |
| idempotency_key | VARCHAR(36) | | | UUID, 중복 요청 방지 |
| request_source | VARCHAR(10) | | O | 요청 출처: `api` / `batch` / `manual` |
| datasets_count | SMALLINT | | | 수신한 데이터셋 수 |
| total_payload_bytes | BIGINT | | | 전체 페이로드 크기 (bytes) |
| version | SMALLINT | | O | 버전 (재처리 시 증가, 기본 1) |
| created_at | TIMESTAMP | | O | 요청 생성 시각 |
| updated_at | TIMESTAMP | | O | 최종 상태 변경 시각 |
| completed_at | TIMESTAMP | | | 완료 시각 |
| error_code | VARCHAR(50) | | | 오류 발생 시 코드 |
| error_message | TEXT | | | 오류 상세 메시지 |

**인덱스**:
- `idx_inp_req_applicant_date` UNIQUE (applicant_id, tax_year, taxpayer_id, EXTRACT(DATE FROM created_at), seq_no) — 동일 날짜 중복 방지
- `idx_inp_req_idempotency` UNIQUE (idempotency_key, applicant_id, tax_year) — 멱등성 보장
- `idx_inp_req_status` (request_status, created_at) — 상태별 조회

---

### 4.2 INP_RAW_DATASET — JSON 원본 데이터셋 보관

> **핵심**: 세부 입력자료를 JSON 형태의 데이터셋으로 받아서 **원본 그대로** 보관하는 테이블.
> INSERT ONLY — UPDATE/DELETE 트리거로 차단.

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| category | VARCHAR(30) | PK | O | 데이터셋 카테고리 코드 (아래 32개 참조) |
| raw_json | JSONB | | O | 원본 JSON 데이터 (수정 불가) |
| schema_version | VARCHAR(10) | | O | JSON 스키마 버전 (예: `1.0`) |
| byte_size | INTEGER | | | JSON 크기 (bytes) |
| checksum | VARCHAR(64) | | | SHA-256 해시 (무결성 검증) |
| received_at | TIMESTAMP | | O | 수신 시각 |

**카테고리 코드 (32개)**:

| 구분 | 카테고리 코드 | 설명 | 필수/선택 |
|------|-------------|------|:--------:|
| **공통 (8)** | `employee_detail` | 근로자 상세 정보 | 선택 |
| | `employee_monthly` | 월별 상시근로자 현황 | 선택 |
| | `investment` | 투자자산 명세 | 선택 |
| | `startup` | 창업/감면 정보 | 선택 |
| | `sme_special` | 중소기업 특별감면 정보 | 선택 |
| | `rd_expense` | R&D 비용 명세 | 선택 |
| | `existing_deduction` | 기존 신고 적용 공제/감면 내역 | 선택 |
| | `foreign_tax` | 외국납부세액 | 선택 |
| **법인 전용 (16)** | `corp_basic` | 법인 기본 정보 | **CORP 필수** |
| | `representative` | 대표이사 정보 | 선택 |
| | `loss_carryforward` | 이월결손금 내역 | 선택 |
| | `credit_carryforward` | 이월 미공제 세액 내역 | 선택 |
| | `branch_location` | 지점 소재지 정보 | 선택 |
| | `interim_tax` | 중간예납 납부 내역 | 선택 |
| | `dividend_income` | 수입배당금 내역 | 선택 |
| | `business_vehicle` | 업무용승용차 내역 | 선택 |
| | `tax_adjustment` | 세무조정 내역 | 선택 |
| | `entertainment` | 접대비 내역 | 선택 |
| | `government_subsidy` | 정부보조금 내역 | 선택 |
| | `shareholder_loan` | 가지급금 내역 | 선택 |
| | `non_business_asset` | 비사업용 자산 | 선택 |
| | `disaster_loss` | 재해손실 내역 | 선택 |
| | `consolidated_sub` | 연결납세 자회사 정보 | 선택 |
| | `depreciation_adjust` | 감가상각 시부인 내역 | 선택 |
| **개인 전용 (8)** | `inc_basic` | 개인사업자 기본 정보 | **INC 필수** |
| | `inc_business` | 사업장별 정보 | 선택 |
| | `inc_other_income` | 기타소득 내역 | 선택 |
| | `inc_deduction` | 소득공제 내역 | 선택 |
| | `inc_sincerity` | 성실신고확인 관련 | 선택 |
| | `inc_foreign_tax` | 개인 외국납부세액 | 선택 |
| | `inc_rental_reduction` | 착한임대인 내역 | 선택 |
| | `inc_joint_biz` | 공동사업자 정보 | 선택 |

---

### 4.3 INP_S_EMPLOYEE_DETAIL — 근로자 상세 (공통, 파싱)

> `employee_detail` 카테고리 JSON에서 파싱한 정규 데이터

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| employee_seq | INTEGER | PK | O | 근로자 일련번호 |
| employee_name | VARCHAR(50) | | | 성명 (암호화) |
| birth_date | DATE | | O | 생년월일 (청년 판단) |
| gender | CHAR(1) | | | M/F |
| hire_date | DATE | | O | 입사일 (근로계약 체결일) |
| termination_date | DATE | | | 퇴직일 |
| contract_type | VARCHAR(10) | | O | REGULAR(정규직)/TEMPORARY(비정규)/DISPATCH(파견)/DAILY(일용) |
| weekly_hours | DECIMAL(5,1) | | | 주당 소정근로시간 |
| contract_months | INTEGER | | | 근로계약기간 (월) |
| is_executive | BOOLEAN | | O | 등기임원 여부 (CORP) |
| is_major_shareholder_related | BOOLEAN | | O | 최대주주/배우자/4촌이내 여부 |
| is_owner_related | BOOLEAN | | O | 대표자/배우자/직계존비속 여부 (INC) |
| youth_category | VARCHAR(20) | | | YOUTH(청년)/DISABLED(장애인)/SENIOR60(60세↑)/CAREER_BREAK(경력단절)/DEFECTOR(북한이탈)/NONE |
| military_months | INTEGER | | | 병역이행 기간 (월, 최대 72) |
| disability_grade | VARCHAR(10) | | | 장애등급 |
| career_break_quit_date | DATE | | | 경력단절 퇴직일 |
| career_break_rehire_date | DATE | | | 경력단절 재고용일 |
| career_break_reason | VARCHAR(20) | | | 퇴직사유 코드 |
| is_converted_regular | BOOLEAN | | | 정규직 전환 여부 |
| regular_conversion_date | DATE | | | 정규직 전환일 |
| parental_leave_return_date | DATE | | | 육아휴직 복귀일 |
| parental_leave_6month_retained | BOOLEAN | | | 복귀 후 6개월 유지 여부 |
| social_insurance_employer_amount | BIGINT | | | 사업주 부담 4대보험 합계 (원) |
| resign_exception_code | VARCHAR(10) | | | 퇴사 예외사유 코드 (추징 면제) |

---

### 4.4 INP_S_EMPLOYEE_MONTHLY — 월별 상시근로자 (공통, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| period_type | VARCHAR(7) | PK | O | `PRIOR` (직전) / `CURRENT` (당해) |
| year_month | CHAR(6) | PK | O | `YYYYMM` |
| total_count | INTEGER | | O | 전체 근로자 수 |
| excluded_count | INTEGER | | O | 제외 대상 수 |
| regular_count | INTEGER | | O | 상시근로자 수 (total - excluded) |
| youth_count | INTEGER | | | 청년등 근로자 수 |
| general_count | INTEGER | | | 일반 근로자 수 |

---

### 4.5 INP_S_INVESTMENT — 투자자산 명세 (공통, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| asset_seq | INTEGER | PK | O | 자산 일련번호 |
| asset_type | VARCHAR(20) | | O | MACHINERY/EQUIPMENT/SOFTWARE/IP_RIGHTS/VEHICLE 등 |
| facility_class | VARCHAR(20) | | O | GENERAL(일반)/NEW_GROWTH(신성장)/NATIONAL_STRATEGIC(국가전략) |
| acquisition_date | DATE | | O | 취득일 |
| acquisition_cost | BIGINT | | O | 취득가액 (원) |
| location_code | VARCHAR(10) | | O | 자산 설치 소재지 행정구역코드 |
| capital_zone | VARCHAR(20) | | | OVER_CONCENTRATION(과밀억제)/GROWTH_MGMT(성장관리)/NON_CAPITAL(비수도권)/DEPOPULATION(인구감소) |
| is_used_asset | BOOLEAN | | O | 중고품 여부 |
| is_leased | BOOLEAN | | O | 리스(금융리스 포함) 여부 |
| is_rental_asset | BOOLEAN | | O | 임대용 자산 여부 (2025년 이후 제외) |
| is_excluded | BOOLEAN | | | 제외 대상 여부 (계산 시 결정) |
| exclusion_reason | VARCHAR(50) | | | 제외 사유 |

---

### 4.6 INP_S_INVESTMENT_HISTORY — 직전 3년 투자 이력 (공통, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| history_year | SMALLINT | PK | O | 투자 연도 (직전 1~3년) |
| total_investment_amount | BIGINT | | O | 해당 연도 총 투자금액 |

---

### 4.7 INP_C_CORP_BASIC — 법인 기본 정보 (CORP 전용, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| corp_name | VARCHAR(100) | | O | 법인명 |
| corp_reg_no | VARCHAR(20) | | O | 법인등록번호 (암호화) |
| biz_reg_no | VARCHAR(12) | | O | 사업자등록번호 |
| corp_size | VARCHAR(10) | | O | SMALL(소기업)/MEDIUM(중기업)/MID_LARGE(중견)/LARGE(대기업) |
| main_industry_code | VARCHAR(10) | | O | KSIC 주업종 코드 |
| sub_industry_code | VARCHAR(10) | | | KSIC 부업종 코드 |
| hq_location_code | VARCHAR(10) | | O | 본점 소재지 행정구역코드 |
| hq_capital_zone | VARCHAR(20) | | O | 본점 수도권 구분 |
| is_depopulation_area | BOOLEAN | | O | 인구감소지역 여부 |
| fiscal_year_start | DATE | | O | 사업연도 개시일 |
| fiscal_year_end | DATE | | O | 사업연도 종료일 |
| total_income | BIGINT | | O | 각 사업연도 소득 |
| loss_deduction | BIGINT | | | 이월결손금 공제액 |
| tax_base | BIGINT | | O | 과세표준 |
| calculated_tax | BIGINT | | O | 산출세액 |
| determined_tax | BIGINT | | O | 결정세액 |
| paid_tax | BIGINT | | O | 납부세액 |
| interim_tax_total | BIGINT | | | 중간예납 세액 합계 |
| withholding_tax | BIGINT | | | 원천징수 세액 |
| claim_reason | TEXT | | | 경정청구 사유 |
| is_venture | BOOLEAN | | | 벤처기업 인정 여부 |
| venture_confirmed_date | DATE | | | 벤처기업 확인일 |

---

### 4.8 INP_C_LOSS_CARRYFORWARD — 이월결손금 (CORP 전용, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| loss_year | SMALLINT | PK | O | 결손금 발생 사업연도 |
| original_loss_amount | BIGINT | | O | 최초 발생 결손금 |
| already_deducted | BIGINT | | O | 기 공제된 금액 |
| remaining_balance | BIGINT | | O | 이월 잔액 |
| expiry_year | SMALLINT | | O | 이월 만료 연도 (2020~ 발생: +15년, 이전: +10년) |

---

### 4.9 INP_C_CREDIT_CARRYFORWARD — 이월 미공제 세액 (CORP 전용, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| provision_code | VARCHAR(10) | PK | O | 조항 코드 (SS24, SS29의8, SS10 등) |
| origin_year | SMALLINT | PK | O | 발생 연도 |
| original_amount | BIGINT | | O | 최초 공제 미사용 금액 |
| already_used | BIGINT | | O | 기 이월 사용 금액 |
| remaining_amount | BIGINT | | O | 이월 잔액 |
| expiry_year | SMALLINT | | O | 이월 만료 연도 (+10년) |

---

### 4.10 INP_I_INC_BASIC — 개인사업자 기본 정보 (INC 전용, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| owner_name | VARCHAR(50) | | O | 대표자명 (암호화) |
| owner_birth_date | DATE | | O | 대표자 생년월일 |
| owner_gender | CHAR(1) | | O | M/F |
| owner_military_months | INTEGER | | | 병역이행 기간 (월) |
| biz_reg_no | VARCHAR(12) | | O | 사업자등록번호 |
| corp_size | VARCHAR(10) | | O | SMALL(소기업)/MEDIUM(중기업) — 개인은 중견/대기업 해당 없음 |
| bookkeeping_type | VARCHAR(20) | | O | DOUBLE(복식부기)/SIMPLE(간편장부)/ESTIMATE(추계) |
| is_sincerity_target | BOOLEAN | | | 성실신고확인대상 여부 |
| filing_deadline | DATE | | | 법정신고기한 (5.31 / 성실 6.30) |
| total_comprehensive_income | BIGINT | | O | 종합소득금액 |
| business_income | BIGINT | | O | 사업소득금액 |
| employment_income | BIGINT | | | 근로소득금액 |
| financial_income | BIGINT | | | 금융소득금액 (2천만 초과 시 합산) |
| pension_income | BIGINT | | | 연금소득금액 |
| other_income | BIGINT | | | 기타소득금액 |
| total_deduction | BIGINT | | | 소득공제 합계 |
| tax_base | BIGINT | | O | 과세표준 |
| calculated_tax | BIGINT | | O | 산출세액 |
| determined_tax | BIGINT | | O | 결정세액 |
| paid_tax | BIGINT | | O | 납부세액 |
| interim_tax | BIGINT | | | 중간예납 세액 |
| withholding_tax | BIGINT | | | 원천징수 세액 |
| claim_reason | TEXT | | | 경정청구 사유 |

---

### 4.11 INP_I_BUSINESS — 사업장별 정보 (INC 전용, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| biz_place_seq | INTEGER | PK | O | 사업장 일련번호 |
| biz_place_name | VARCHAR(100) | | O | 상호 |
| biz_reg_no | VARCHAR(12) | | O | 사업자등록번호 |
| industry_code | VARCHAR(10) | | O | KSIC 업종코드 |
| location_code | VARCHAR(10) | | O | 사업장 소재지 행정구역코드 |
| capital_zone | VARCHAR(20) | | O | 수도권 구분 |
| is_depopulation_area | BOOLEAN | | O | 인구감소지역 여부 |
| startup_date | DATE | | | 개업일 |
| business_income | BIGINT | | O | 사업장별 소득금액 |
| is_joint_business | BOOLEAN | | O | 공동사업자 여부 |
| profit_share_ratio | DECIMAL(5,4) | | | 손익분배비율 (공동사업 시) |

---

### 4.12 INP_S_RD_EXPENSE — R&D 비용 명세 (공통, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| rd_year | SMALLINT | PK | O | R&D 비용 발생 연도 |
| rd_type | VARCHAR(20) | PK | O | GENERAL(일반)/NEW_GROWTH(신성장)/NATIONAL_STRATEGIC(국가전략) |
| labor_cost | BIGINT | | O | 연구전담 인건비 |
| material_cost | BIGINT | | | 연구 재료비 |
| outsourcing_cost | BIGINT | | | 위탁연구개발비 |
| depreciation_cost | BIGINT | | | 연구용 감가상각비 |
| total_rd_cost | BIGINT | | O | 해당 연도 R&D비 합계 |
| has_rd_department | BOOLEAN | | O | 연구전담부서 보유 여부 |
| koita_registered | BOOLEAN | | | KOITA 등록 여부 |

---

### 4.13 INP_S_EXISTING_DEDUCTION — 기존 신고 공제/감면 내역 (공통, 파싱)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| provision_code | VARCHAR(10) | PK | O | 적용 조항 코드 |
| applied_amount | BIGINT | | O | 기존 신고 시 적용 금액 |
| is_exemption | BOOLEAN | | O | 감면(true) / 공제(false) |
| carryforward_amount | BIGINT | | | 이월 잔액 |

---

### 4.14 기타 입력 테이블 (요약)

| 테이블명 | 세목 | 주요 컬럼 | 원본 카테고리 |
|---------|------|----------|-------------|
| INP_C_REPRESENTATIVE | CORP | 대표이사 성명/생년월일/청년여부/병역 | representative |
| INP_C_BRANCH_LOCATION | CORP | 지점별 소재지/수도권구분 | branch_location |
| INP_C_INTERIM_TAX | CORP | 중간예납 건별 납부일/납부액 | interim_tax |
| INP_C_DIVIDEND_INCOME | CORP | 수입배당금 국가/지분율/금액 | dividend_income |
| INP_C_TAX_ADJUSTMENT | CORP | 접대비/가지급금/업무용승용차 조정 | tax_adjustment |
| INP_C_DEPRECIATION | CORP | 자산별 감가상각 시부인 이력 | depreciation_adjust |
| INP_I_OTHER_INCOME | INC | 소득종류별 금액/분리과세여부 | inc_other_income |
| INP_I_DEDUCTION | INC | 인적/연금/노란우산/건보 공제 | inc_deduction |
| INP_I_SINCERITY | INC | 성실신고확인비용/의료비/교육비 | inc_sincerity |
| INP_I_RENTAL_REDUCTION | INC | 임차인정보/인하액/인하기간 | inc_rental_reduction |
| INP_I_JOINT_BIZ | INC | 공동사업장 전체소득/지분율 | inc_joint_biz |

---

## 5. Layer 2: 중간 요약 테이블

> **목적**: 세부 입력자료(INP_)를 바탕으로 기준정보(REF_)를 활용하여 환급액 계산에 활용할 수 있도록 **계산 엔진 입력 형태로 요약**한 중간 데이터.
> INP_RAW_DATASET에서 언제든 재생성 가능 (파생 데이터).

### 5.1 SMR_BASIC_INFO — 기초정보 요약

> INP_C_CORP_BASIC 또는 INP_I_INC_BASIC + REF_* 기준정보를 결합하여 생성

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| tax_type | VARCHAR(4) | | O | CORP / INC |
| corp_size | VARCHAR(10) | | O | 기업 규모 (REF_S_SME_CRITERIA 활용 검증) |
| main_industry_code | VARCHAR(10) | | O | KSIC 주업종 |
| industry_eligible | BOOLEAN | | O | 감면 대상 업종 여부 (REF_S_INDUSTRY_ELIGIBILITY 참조) |
| hq_capital_zone | VARCHAR(20) | | O | 본점/주사업장 수도권 구분 (REF_S_CAPITAL_ZONE 참조) |
| is_depopulation_area | BOOLEAN | | O | 인구감소지역 (REF_S_DEPOPULATION_AREA 참조) |
| effective_law_year | SMALLINT | | O | 적용 법령 연도 (MX-02 결정) |
| tax_base | BIGINT | | O | 과세표준 |
| calculated_tax | BIGINT | | O | 산출세액 |
| determined_tax | BIGINT | | O | 결정세액 |
| paid_tax | BIGINT | | O | 총 납부세액 |
| claim_deadline | DATE | | O | 경정청구 마감일 (법정신고기한 + 5년) |
| is_within_deadline | BOOLEAN | | O | 경정청구 기한 내 여부 |
| bookkeeping_type | VARCHAR(20) | | | INC 전용: 장부유형 |
| is_sincerity_target | BOOLEAN | | | INC 전용: 성실신고확인대상 |
| comprehensive_income | BIGINT | | | INC 전용: 종합소득금액 |
| is_owner_youth | BOOLEAN | | | 대표자 청년 여부 (창업감면 판단) |
| created_at | TIMESTAMP | | O | 생성 시각 |

---

### 5.2 SMR_EMPLOYEE — 고용 현황 요약

> INP_S_EMPLOYEE_DETAIL + INP_S_EMPLOYEE_MONTHLY 파싱 → 제외 대상 필터링 → 연평균 산정

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| period_type | VARCHAR(7) | PK | O | `PRIOR` / `CURRENT` |
| total_annual_avg | DECIMAL(10,2) | | O | 전체 상시근로자 연평균 (소수점 3자리 절사) |
| youth_annual_avg | DECIMAL(10,2) | | O | 청년등 상시근로자 연평균 |
| general_annual_avg | DECIMAL(10,2) | | O | 일반 상시근로자 연평균 |
| excluded_count | INTEGER | | O | 제외 인원 (임원/특수관계/단시간 등) |
| career_break_count | INTEGER | | | 경력단절여성 재고용 인원 |
| converted_regular_count | INTEGER | | | 정규직 전환 인원 |
| parental_return_count | INTEGER | | | 육아휴직 복귀 인원 |
| youth_increase | DECIMAL(10,2) | | | 청년등 증가분 (CURRENT - PRIOR) |
| general_increase | DECIMAL(10,2) | | | 일반 증가분 |
| total_increase | DECIMAL(10,2) | | | 전체 증가분 |
| social_insurance_total | BIGINT | | | 사업주 부담 4대보험 합계 |

**산정 공식**:
```
연평균 = TRUNCATE(SUM(월별 상시근로자) / 해당월수, 2)  -- 소수점 3자리 절사
청년 판단: 2024 이전=과세연도 말일 기준, 2025 이후=근로계약 체결일 기준
```

---

### 5.3 SMR_DEDUCTION_ITEM — 공제/감면 항목별 요약

> 각 점검항목에 대해 입력자료를 기준정보와 결합하여 **계산 가능 상태**로 요약

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| provision_code | VARCHAR(10) | PK | O | 조항 코드: SS24, SS29_8, SS6, SS7, SS10, SS30_4 등 |
| deduction_type | VARCHAR(10) | | O | EXEMPTION(감면) / CREDIT(공제) / DEDUCTION(소득공제) |
| is_eligible | BOOLEAN | | O | 적격 여부 (사전점검 결과) |
| ineligible_reason | VARCHAR(100) | | | 부적격 사유 |
| base_amount | BIGINT | | | 기초 금액 (투자금액/R&D비/보험료 등) |
| applicable_rate | DECIMAL(5,4) | | | 적용 공제율/감면율 (REF_* 참조 결정) |
| rate_detail_json | JSONB | | | 공제율 결정 근거 상세 (복수 요율 시) |
| limit_amount | BIGINT | | | 공제/감면 한도 |
| carryforward_balance | BIGINT | | | 이월 미공제 잔액 |
| is_min_tax_subject | BOOLEAN | | O | 최저한세 적용 대상 여부 |
| min_tax_exempt_rate | DECIMAL(3,2) | | | 최저한세 배제율 (R&D: 0.5/1.0) |
| agricultural_tax_type | VARCHAR(10) | | O | EXEMPT(비과세) / TAXABLE_20(과세 20%) |
| is_carryforward_eligible | BOOLEAN | | O | 이월공제 가능 여부 |
| application_priority | SMALLINT | | O | 적용순서 (1=감면, 2=이월불가공제, 3=이월가능공제) |
| tax_type_scope | VARCHAR(4) | | O | CORP / INC / BOTH |
| source_input_category | VARCHAR(30) | | O | 원본 INP 카테고리 |
| validation_status | VARCHAR(10) | | O | VALID / WARNING / ERROR |
| validation_message | TEXT | | | 검증 메시지 |

---

### 5.4 SMR_FINANCIAL — 재무/세무 수치 요약

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| tax_base_original | BIGINT | | O | 기존 신고 과세표준 |
| calculated_tax_original | BIGINT | | O | 기존 산출세액 |
| determined_tax_original | BIGINT | | O | 기존 결정세액 |
| paid_tax_total | BIGINT | | O | 총 납부세액 (본세+중간예납+원천) |
| loss_cf_total_balance | BIGINT | | | 이월결손금 총 잔액 |
| loss_cf_applicable | BIGINT | | | 적용 가능 이월결손금 (기한 내, FIFO) |
| credit_cf_total_balance | BIGINT | | | 이월 미공제 세액 총 잔액 |
| existing_exemption_total | BIGINT | | | 기존 적용 감면 합계 |
| existing_credit_total | BIGINT | | | 기존 적용 공제 합계 |
| minimum_tax_amount | BIGINT | | | 최저한세 금액 (CORP: 과세표준×세율, INC: 산출세액×35/45%) |
| available_credit_limit | BIGINT | | | 공제 가능 한도 (산출세액 - 최저한세) |
| prior_year_tax_base | BIGINT | | | 직전연도 과세표준 (소급공제 계산용) |
| prior_year_calculated_tax | BIGINT | | | 직전연도 산출세액 |
| prior_year_paid_tax | BIGINT | | | 직전연도 납부세액 |
| inc_total_deduction | BIGINT | | | INC: 소득공제 합계 |
| inc_comprehensive_income | BIGINT | | | INC: 종합소득금액 |

---

### 5.5 SMR_ELIGIBILITY — 적격성 판정 결과

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| check_code | VARCHAR(10) | PK | O | 점검 코드 (M3-00~06, P3-01~04) |
| check_name | VARCHAR(100) | | O | 점검항목명 |
| result | VARCHAR(10) | | O | PASS / HARD_FAIL / SOFT_FAIL / WARNING / SKIP |
| detail_message | TEXT | | | 상세 결과 메시지 |
| legal_basis | VARCHAR(50) | | | 관련 법령 |
| checked_at | TIMESTAMP | | O | 점검 시각 |

---

### 5.6 SMR_PREP — 사전준비 요약

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| effective_law_version | VARCHAR(20) | | O | 적용 법령 버전 (연도+개정차수) |
| tax_rate_snapshot_json | JSONB | | O | 적용 세율 스냅샷 (과세연도 기준) |
| min_tax_rate_snapshot_json | JSONB | | O | 최저한세율 스냅샷 |
| candidate_provisions | TEXT[] | | O | 적용 가능 공제/감면 후보 조항 목록 |
| candidate_count | SMALLINT | | O | 후보 수 |
| corp_size_determined | VARCHAR(10) | | O | 최종 판정 기업 규모 |
| capital_zone_determined | VARCHAR(20) | | O | 최종 소재지 구분 |
| prepared_at | TIMESTAMP | | O | 준비 완료 시각 |

---

### 5.7 SMR_VALIDATION_LOG — 검증 결과 로그

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| rule_code | VARCHAR(10) | PK | O | 규칙 코드 (V01~W06, VP01~XP05, 총 107개) |
| rule_type | VARCHAR(2) | | O | V/C/B/L/X/W/VP/CP/BP/LP/WP/XP |
| severity | VARCHAR(10) | | O | ERROR / WARNING |
| result | VARCHAR(10) | | O | PASS / FAIL |
| detail_message | TEXT | | | 상세 메시지 |
| validated_at | TIMESTAMP | | O | 검증 시각 |

---

## 6. Layer 3: 산출 결과 / 출력 보관 테이블

> **목적**: 계산 엔진의 산출 결과를 정규 테이블에 보관하고, **JSON 형태의 데이터셋으로 변환하기 위한** 출력 보관 역할을 한다.
> 8섹션 JSON 보고서는 이 테이블들에서 직렬화된다.

### 6.1 OUT_CREDIT_DETAIL — 개별 공제/감면 산출 결과

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| provision_code | VARCHAR(10) | PK | O | 조항 코드 |
| module_code | VARCHAR(10) | | O | 산출 모듈 (M4-01, P4-01 등) |
| credit_type | VARCHAR(10) | | O | EXEMPTION / CREDIT / DEDUCTION |
| gross_amount | BIGINT | | O | 공제/감면 산출 총액 (절사 전) |
| truncated_amount | BIGINT | | O | 절사 후 금액 (10원 미만 절사) |
| min_tax_subject_amount | BIGINT | | | 최저한세 적용 대상 금액 |
| min_tax_exempt_amount | BIGINT | | | 최저한세 배제 금액 (R&D 등) |
| agricultural_tax_amount | BIGINT | | | 농특세 금액 (과세 시 20%) |
| net_amount | BIGINT | | O | 실질 공제/감면 금액 (= truncated - agricultural_tax) |
| carryforward_current | BIGINT | | | 이월 미사용분 (당기 발생) |
| carryforward_used | BIGINT | | | 이월분 사용액 (기존 이월 소진) |
| is_applied | BOOLEAN | | O | 최종 적용 여부 (최적조합 반영 후) |
| not_applied_reason | VARCHAR(50) | | | 미적용 사유 (배제/한도초과/부적격) |
| calculation_detail_json | JSONB | | O | 계산 상세 (공식, 입력값, 중간값) |
| legal_basis | VARCHAR(50) | | O | 근거 법령 |
| calculated_at | TIMESTAMP | | O | 산출 시각 |

---

### 6.2 OUT_COMBINATION — 최적 조합 결과

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| combo_rank | SMALLINT | PK | O | 조합 순위 (1=최적, 2=차선...) |
| included_provisions | TEXT[] | | O | 포함된 조항 코드 목록 |
| total_exemption | BIGINT | | O | 감면 합계 |
| total_credit | BIGINT | | O | 공제 합계 |
| total_agricultural_tax | BIGINT | | O | 농특세 합계 |
| net_benefit | BIGINT | | O | 실질 환급 효과 (감면+공제-농특세) |
| min_tax_applied | BIGINT | | O | 최저한세 적용 금액 |
| refund_estimate | BIGINT | | O | 환급 예상액 |
| application_order_json | JSONB | | O | 적용순서 상세 (감면→이월불가→이월가능) |
| convergence_iterations | SMALLINT | | | 순환참조 수렴 반복 횟수 (MX-01) |
| algorithm_type | VARCHAR(10) | | O | BRANCH_BOUND / GREEDY |
| search_time_ms | INTEGER | | | 탐색 소요 시간 (ms) |
| is_selected | BOOLEAN | | O | 최종 선택된 조합 여부 |

---

### 6.3 OUT_EXCLUSION_VERIFY — 상호배제 적용 내역

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| combo_rank | SMALLINT | PK | O | 조합 순위 |
| exclusion_rule_id | INTEGER | PK | O | 상호배제 규칙 번호 (1~10) |
| provision_a | VARCHAR(10) | | O | 조항 A |
| provision_b | VARCHAR(10) | | O | 조항 B |
| rule_result | VARCHAR(10) | | O | ALLOWED / BLOCKED / CONDITIONAL |
| year_condition | VARCHAR(50) | | | 연도별 조건 (예: "2024 이전: 허용, 2025 이후: 택1") |
| selected_provision | VARCHAR(10) | | | CONDITIONAL 시 선택된 조항 |
| selection_reason | TEXT | | | 선택 근거 (금액 비교 등) |

---

### 6.4 OUT_REFUND — 환급 결과

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| tax_base_revised | BIGINT | | O | 경정 후 과세표준 |
| calculated_tax_revised | BIGINT | | O | 경정 후 산출세액 |
| total_exemption | BIGINT | | O | 적용 감면 합계 |
| total_credit | BIGINT | | O | 적용 공제 합계 |
| determined_tax_revised | BIGINT | | O | 경정 후 결정세액 |
| main_tax_refund | BIGINT | | O | 본세 환급액 (= 기존납부 - 경정후결정) |
| refund_interest_main | BIGINT | | | 환급가산금 (본세분) |
| refund_interest_interim | BIGINT | | | 환급가산금 (중간예납분, CORP) |
| local_tax_refund | BIGINT | | | 지방소득세 환급액 (본세 × 10%) |
| agricultural_tax_change | BIGINT | | | 농특세 변동액 |
| total_expected_receipt | BIGINT | | O | 총 수령 예상액 |
| refund_interest_rate | DECIMAL(5,4) | | | 적용 환급가산금 이율 |
| interest_start_date | DATE | | | 기산일 |
| interest_end_date | DATE | | | 종료일 (환급 결정일) |
| interest_days | INTEGER | | | 경과일수 |
| comparison_json | JSONB | | O | 기존 vs 경정후 비교표 JSON |

---

### 6.5 OUT_RISK — 사후관리 리스크

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| risk_seq | INTEGER | PK | O | 리스크 일련번호 |
| risk_type | VARCHAR(30) | | O | EMPLOYEE_DECREASE(고용감소)/ASSET_DISPOSAL(자산처분)/VENTURE_LOSS(벤처해제) 등 |
| provision_code | VARCHAR(10) | | O | 관련 조항 |
| scenario_desc | TEXT | | O | 시나리오 설명 |
| principal_surcharge | BIGINT | | O | 본세 추징 예상액 |
| interest_surcharge | BIGINT | | | 이자상당가산액 |
| total_surcharge | BIGINT | | O | 총 추징 예상액 |
| monitoring_period_months | INTEGER | | O | 사후관리 기간 (월) |
| monitoring_start_date | DATE | | | 모니터링 시작일 |
| monitoring_end_date | DATE | | | 모니터링 종료일 |
| risk_level | VARCHAR(10) | | O | HIGH / MEDIUM / LOW |

---

### 6.6 OUT_REPORT_JSON — JSON 보고서 직렬화

> **목적**: 산출 결과를 JSON 형태의 데이터셋으로 **변환하여 보관**하는 최종 출력 테이블

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| section_a_json | JSONB | | O | 해당가능성 진단 (SMR_ELIGIBILITY + SMR_BASIC_INFO 직렬화) |
| section_b_json | JSONB | | O | 후보 항목 리스트 (OUT_CREDIT_DETAIL 직렬화) |
| section_c_json | JSONB | | O | 최적조합/배제 (OUT_COMBINATION + OUT_EXCLUSION_VERIFY 직렬화) |
| section_d_json | JSONB | | O | 환급/추징 금액 (OUT_REFUND 직렬화) |
| section_e_json | JSONB | | O | 사후관리/증빙목록 (OUT_RISK 직렬화) |
| section_f_json | JSONB | | O | 이월분석/소급공제NPV |
| section_g_json | JSONB | | O | 제출 데이터/메타 정보 |
| section_h_json | JSONB | | O | 산출진행로그/검증결과 (SMR_VALIDATION_LOG 직렬화) |
| report_version | VARCHAR(10) | | O | 보고서 스키마 버전 |
| generated_at | TIMESTAMP | | O | 생성 시각 |
| total_byte_size | INTEGER | | | 전체 JSON 크기 |

---

### 6.7 세목 전용 출력 테이블

#### OUT_I_LOSS_CARRYBACK — 개인 소급공제 결과 (INC 전용)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| prior_year_tax_base | BIGINT | | O | 직전연도 과세표준 |
| prior_year_calculated_tax | BIGINT | | O | 직전연도 산출세액 |
| current_year_loss | BIGINT | | O | 당해연도 결손금 |
| carryback_refund | BIGINT | | O | 소급공제 환급액 |
| carryforward_npv | BIGINT | | | 이월공제 NPV (비교용) |
| recommendation | VARCHAR(10) | | O | CARRYBACK / CARRYFORWARD |
| recommendation_reason | TEXT | | | 권고 근거 |

#### OUT_I_INCOME_DEDUCTION — 개인 소득공제 최적화 (INC 전용)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| deduction_item_seq | INTEGER | PK | O | 공제 항목 일련번호 |
| deduction_type | VARCHAR(30) | | O | PERSONAL/PENSION/INSURANCE/YELLOW_UMBRELLA/HOUSING 등 |
| original_amount | BIGINT | | | 기존 신고 적용 금액 |
| optimized_amount | BIGINT | | O | 최적화 후 적용 금액 |
| difference | BIGINT | | O | 변경 금액 |
| tax_bracket_before | VARCHAR(10) | | | 변경 전 세율 구간 |
| tax_bracket_after | VARCHAR(10) | | | 변경 후 세율 구간 |
| tax_saving | BIGINT | | O | 절세 효과 |

#### OUT_C_TAX_ADJUSTMENT — 법인 세무조정 결과 (CORP 전용)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| req_id | VARCHAR(30) | PK | O | FK → INP_REQUEST |
| adjustment_seq | INTEGER | PK | O | 조정 일련번호 |
| adjustment_type | VARCHAR(30) | | O | ENTERTAINMENT/SHAREHOLDER_LOAN/BUSINESS_VEHICLE/DEPRECIATION |
| original_amount | BIGINT | | O | 기존 조정 금액 |
| revised_amount | BIGINT | | O | 경정 후 조정 금액 |
| difference | BIGINT | | O | 변동액 |
| tax_effect | BIGINT | | O | 세액 영향 |
| detail_json | JSONB | | | 조정 상세 |

---

## 7. Layer 4: 기준정보 테이블

> **목적**: req_id에 의존하지 않는 독립 기준정보. 세율, 공제율, 한도, 상호배제 규칙 등.
> 연도별 버전 관리를 통해 경정청구 5년 범위 (2021~2025) 전체 지원.

### 7.1 기준정보 테이블 목록

| 테이블명 | 세목 | 설명 | 갱신 주기 |
|---------|:----:|------|----------|
| **REF_S_TAX_RATE_BRACKET** | 공통 | 법인세/소득세 과세표준별 세율 (2018~2025) | 세법개정 시 (연 1회) |
| **REF_S_MIN_TAX_RATE** | 공통 | 최저한세율 (기업규모/과세표준 구간별) | 조특법 개정 시 |
| **REF_S_MUTUAL_EXCLUSION** | 공통 | 상호배제 10규칙 (연도별 조건 포함) | 조특법 개정 시 |
| **REF_S_CAPITAL_ZONE** | 공통 | 수도권 과밀억제/성장관리/비수도권 구분 | 비정기 (법 개정) |
| **REF_S_DEPOPULATION_AREA** | 공통 | 인구감소지역 목록 | 연 1회 (행안부 고시) |
| **REF_S_INDUSTRY_ELIGIBILITY** | 공통 | KSIC별 감면 대상/제외 업종 | 조특법 개정 시 |
| **REF_S_KSIC_CODE** | 공통 | 한국표준산업분류 코드 | 5년 주기 (통계청) |
| **REF_S_EMPLOYMENT_CREDIT** | 공통 | 통합고용 공제 단가 (규모/유형별) | 시행령 개정 시 |
| **REF_S_REFUND_INTEREST_RATE** | 공통 | 환급가산금 이율 | 분기별 (국세청 고시) |
| **REF_S_NONGTEUKSE** | 공통 | 농특세 과세/비과세 구분 (조항별) | 농특세법 개정 시 |
| **REF_S_LAW_VERSION** | 공통 | 법령 버전 매핑 (과세연도→적용법) | 세법개정 시 |
| **REF_S_EXCHANGE_RATE** | 공통 | 환율 (외국납부세액 계산) | 매 영업일 |
| **REF_S_SYSTEM_PARAM** | 공통 | 시스템 파라미터 (greedy_threshold 등) | 수시 |
| **REF_S_SME_CRITERIA** | 공통 | 중소기업 판정 기준 (매출/독립성/졸업유예) | 중기법 개정 시 |
| **REF_S_SURCHARGE_RATE** | 공통 | 가산세/이자상당가산액 이율 | 국기법 개정 시 |
| **REF_C_INVESTMENT_CREDIT_RATE** | CORP | 투자공제율 (규모/시설/연도별) | 조특법 개정 시 |
| **REF_C_CORP_TAX_RATE** | CORP | 법인세율 상세 (연도별) | 세법개정 시 |
| **REF_C_DEEMED_INTEREST_RATE** | CORP | 인정이자율 | 연 1회 (국세청) |
| **REF_C_STARTUP_EXEMPTION** | CORP | 창업감면율 (소재지/청년/연도별) | 조특법 개정 시 |
| **REF_C_RD_CREDIT_RATE** | CORP | R&D 공제율 (유형/규모별) | 조특법 개정 시 |
| **REF_C_RD_MIN_TAX_EXEMPT** | CORP | R&D 최저한세 배제율 (유형/규모별) | 조특법 개정 시 |
| **REF_C_LOSS_CF_LIMIT** | CORP | 이월결손금 공제 한도율 (규모/연도별) | 법인세법 개정 시 |
| **REF_I_INCOME_TAX_RATE** | INC | 소득세율 상세 (연도별 8단계) | 세법개정 시 |
| **REF_I_SINCERITY_THRESHOLD** | INC | 성실신고확인대상 수입금액 기준 | 소득세법 개정 시 |
| **REF_I_DEDUCTION_LIMIT** | INC | 소득공제 항목별 한도 | 소득세법 개정 시 |
| **REF_I_STARTUP_EXEMPTION** | INC | 개인 창업감면율 (소재지/청년/연도별) | 조특법 개정 시 |

### 7.2 REF_S_TAX_RATE_BRACKET — 세율 테이블 (대표 스키마)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| tax_type | VARCHAR(4) | PK | O | CORP / INC |
| effective_year_from | SMALLINT | PK | O | 적용 시작 연도 |
| effective_year_to | SMALLINT | | O | 적용 종료 연도 (9999=현행) |
| bracket_seq | SMALLINT | PK | O | 구간 순번 |
| lower_bound | BIGINT | | O | 구간 하한 (원) |
| upper_bound | BIGINT | | | 구간 상한 (원, NULL=초과) |
| tax_rate | DECIMAL(5,4) | | O | 세율 |
| progressive_deduction | BIGINT | | O | 누진공제 (원) |

### 7.3 REF_S_MUTUAL_EXCLUSION — 상호배제 규칙 (대표 스키마)

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| rule_id | INTEGER | PK | O | 규칙 번호 (1~10) |
| provision_a | VARCHAR(10) | | O | 조항 A 코드 |
| provision_b | VARCHAR(10) | | O | 조항 B 코드 |
| allowed | VARCHAR(15) | | O | ALLOWED/BLOCKED/CONDITIONAL |
| condition_year_from | SMALLINT | | | 조건 적용 시작 연도 |
| condition_year_to | SMALLINT | | | 조건 적용 종료 연도 |
| condition_detail | TEXT | | | 조건 상세 |
| priority_hint | VARCHAR(10) | | | A / B / COMPARE — 택1 시 우선순위 힌트 |

---

## 8. Layer 5: 감사 로그 테이블

### 8.1 AUD_CALCULATION_LOG — 계산 이력

| 컬럼 | 타입 | PK | 필수 | 설명 |
|------|------|:--:|:---:|------|
| log_id | BIGSERIAL | PK | O | 자동 증가 |
| req_id | VARCHAR(30) | | O | FK → INP_REQUEST |
| module_code | VARCHAR(10) | | O | 실행 모듈 (M1-01, M4-01 등) |
| step_name | VARCHAR(50) | | O | 단계명 |
| input_snapshot_json | JSONB | | | 입력 스냅샷 |
| output_snapshot_json | JSONB | | | 출력 스냅샷 |
| execution_time_ms | INTEGER | | | 실행 시간 (ms) |
| status | VARCHAR(10) | | O | SUCCESS / ERROR / WARNING |
| error_message | TEXT | | | 오류 메시지 |
| created_at | TIMESTAMP | | O | 기록 시각 |
| created_by | VARCHAR(50) | | O | 실행 주체 |

**제약**: APPEND ONLY (INSERT만 허용, UPDATE/DELETE 트리거 차단)

---

## 9. JSON 데이터셋 구조

### 9.1 입력 JSON 데이터셋 구조 (API-01 요청)

```json
{
  "applicant_type": "C",
  "applicant_id": "TAX-00001",
  "tax_type": "CORP",
  "tax_year": 2024,
  "datasets": [
    {
      "category": "corp_basic",
      "schema_version": "1.0",
      "data": {
        "corp_name": "테스트법인",
        "corp_reg_no": "110111-0000001",
        "biz_reg_no": "123-45-67890",
        "corp_size": "SMALL",
        "main_industry_code": "C25110",
        "hq_location_code": "1111000000",
        "fiscal_year_start": "2024-01-01",
        "fiscal_year_end": "2024-12-31",
        "total_income": 500000000,
        "tax_base": 480000000,
        "calculated_tax": 69200000,
        "determined_tax": 65000000,
        "paid_tax": 65000000
      }
    },
    {
      "category": "employee_detail",
      "schema_version": "1.0",
      "data": [
        {
          "employee_seq": 1,
          "birth_date": "1995-03-15",
          "hire_date": "2023-01-02",
          "contract_type": "REGULAR",
          "weekly_hours": 40,
          "is_executive": false,
          "is_major_shareholder_related": false,
          "youth_category": "YOUTH",
          "military_months": 21
        }
      ]
    },
    {
      "category": "investment",
      "schema_version": "1.0",
      "data": [
        {
          "asset_seq": 1,
          "asset_type": "MACHINERY",
          "facility_class": "GENERAL",
          "acquisition_date": "2024-06-15",
          "acquisition_cost": 200000000,
          "location_code": "4113500000",
          "is_used_asset": false,
          "is_leased": false,
          "is_rental_asset": false
        }
      ]
    }
  ]
}
```

### 9.2 출력 JSON 데이터셋 구조 (8섹션 보고서)

```json
{
  "req_id": "C-1234567890-20260218-001",
  "report_version": "1.0",
  "generated_at": "2026-02-18T15:30:00+09:00",
  "sections": {
    "A_eligibility": {
      "tax_type": "CORP",
      "corp_size": "SMALL",
      "within_deadline": true,
      "hard_fail_items": [],
      "warning_items": ["W02: 동일업종 폐업이력 확인 필요"],
      "candidate_count": 5
    },
    "B_credit_candidates": [
      {
        "provision_code": "SS24",
        "provision_name": "통합투자세액공제",
        "gross_amount": 20000000,
        "net_amount": 16000000,
        "agricultural_tax": 4000000,
        "min_tax_subject": true,
        "legal_basis": "조특법 §24"
      }
    ],
    "C_optimal_combination": {
      "selected_rank": 1,
      "combinations": [
        {
          "rank": 1,
          "provisions": ["SS24", "SS29_8", "SS10"],
          "net_benefit": 45000000,
          "algorithm": "BRANCH_BOUND"
        }
      ],
      "exclusion_applied": [
        { "rule_id": 1, "provision_a": "SS6", "provision_b": "SS7", "result": "BLOCKED" }
      ]
    },
    "D_refund": {
      "tax_base_original": 480000000,
      "tax_base_revised": 480000000,
      "determined_tax_original": 65000000,
      "determined_tax_revised": 20000000,
      "main_tax_refund": 45000000,
      "refund_interest": 1200000,
      "local_tax_refund": 4500000,
      "total_expected_receipt": 50700000
    },
    "E_risk": [],
    "F_carryforward_analysis": {},
    "G_metadata": {},
    "H_validation_log": []
  }
}
```

---

## 10. 엔티티 관계도

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          INP_REQUEST (요청 마스터)                            │
│                     PK: req_id                                              │
│                     tax_type: CORP | INC                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                    │                                        │
│           ┌────────────────────────┼────────────────────────┐               │
│           │ 1:N                    │ 1:N                    │ 1:N           │
│           ▼                        ▼                        ▼               │
│  ┌─────────────────┐  ┌──────────────────────┐  ┌─────────────────────┐    │
│  │ INP_RAW_DATASET  │  │ INP_S_EMPLOYEE_DETAIL│  │ INP_S_INVESTMENT    │    │
│  │ (JSON 원본)      │  │ (근로자 상세)         │  │ (투자자산 상세)     │    │
│  │ PK: req+category │  │ PK: req+emp_seq      │  │ PK: req+asset_seq   │    │
│  └─────────────────┘  └──────────────────────┘  └─────────────────────┘    │
│           │                                                                 │
│  CORP ────┼──── INP_C_CORP_BASIC, INP_C_LOSS_CF, INP_C_CREDIT_CF, ...     │
│  INC  ────┼──── INP_I_INC_BASIC, INP_I_BUSINESS, INP_I_DEDUCTION, ...     │
│           │                                                                 │
├───────────┼── Layer 2: 요약 (REF_* 참조하여 파생) ──────────────────────────┤
│           │                                                                 │
│           ▼                                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌───────────────────┐ ┌──────────────┐  │
│  │ SMR_BASIC_   │ │ SMR_EMPLOYEE │ │ SMR_DEDUCTION_    │ │ SMR_FINANCIAL│  │
│  │ INFO         │ │              │ │ ITEM              │ │              │  │
│  │ 1:1          │ │ 1:2 (P/C)   │ │ 1:N (조항별)      │ │ 1:1          │  │
│  └──────────────┘ └──────────────┘ └───────────────────┘ └──────────────┘  │
│  ┌──────────────┐ ┌──────────────┐ ┌───────────────────┐                   │
│  │ SMR_ELIGIBI- │ │ SMR_PREP     │ │ SMR_VALIDATION_   │                   │
│  │ LITY         │ │              │ │ LOG               │                   │
│  │ 1:N (점검별) │ │ 1:1          │ │ 1:N (규칙별)      │                   │
│  └──────────────┘ └──────────────┘ └───────────────────┘                   │
│           │                                                                 │
├───────────┼── Layer 3: 출력 (JSON 직렬화 대상) ─────────────────────────────┤
│           │                                                                 │
│           ▼                                                                 │
│  ┌──────────────────┐ ┌──────────────┐ ┌──────────────────────────────┐    │
│  │ OUT_CREDIT_DETAIL│ │ OUT_COMBI-   │ │ OUT_EXCLUSION_VERIFY          │    │
│  │ 1:N (조항별)     │ │ NATION       │ │ N:N (조합×규칙)               │    │
│  │                  │ │ 1:N (순위별) │ │                               │    │
│  └──────────────────┘ └──────────────┘ └──────────────────────────────┘    │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────────────────┐    │
│  │ OUT_REFUND   │ │ OUT_RISK     │ │ OUT_REPORT_JSON                   │    │
│  │ 1:1          │ │ 1:N          │ │ 1:1 (8섹션 JSONB)                │    │
│  └──────────────┘ └──────────────┘ └──────────────────────────────────┘    │
│           │                                                                 │
│  CORP ────┼──── OUT_C_TAX_ADJUSTMENT                                       │
│  INC  ────┼──── OUT_I_LOSS_CARRYBACK, OUT_I_INCOME_DEDUCTION               │
│                                                                             │
├── Layer 4: 기준정보 (독립, req_id 없음) ────────────────────────────────────┤
│                                                                             │
│  REF_S_TAX_RATE_BRACKET    REF_S_MUTUAL_EXCLUSION    REF_S_CAPITAL_ZONE    │
│  REF_S_MIN_TAX_RATE        REF_S_EMPLOYMENT_CREDIT   REF_S_INDUSTRY_*      │
│  REF_C_* (법인 전용 7개)   REF_I_* (개인 전용 4개)   REF_S_* (공통 15개)   │
│                                                                             │
├── Layer 5: 감사 (APPEND ONLY) ──────────────────────────────────────────────┤
│                                                                             │
│  AUD_CALCULATION_LOG  (전 모듈 계산 이력)                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. 테이블 요약 매트릭스

### 11.1 계층별 테이블 수

| 계층 | 접두어 | 공통 (S) | 법인 전용 (C) | 개인 전용 (I) | 합계 |
|------|--------|:--------:|:------------:|:------------:|:----:|
| L1: 입력 | INP_ | 6 | 8 | 6 | **20** |
| L2: 요약 | SMR_ | 7 | 0 | 0 | **7** |
| L3: 출력 | OUT_ | 6 | 1 | 2 | **9** |
| L4: 기준정보 | REF_ | 15 | 7 | 4 | **26** |
| L5: 감사 | AUD_ | 1 | 0 | 0 | **1** |
| **합계** | | **35** | **16** | **12** | **63** |

### 11.2 참고1번 대비 변경점

| 항목 | 참고1번 (초안) | 본 스키마 (v1.0) | 변경 이유 |
|------|-------------|-----------------|----------|
| 테이블 수 | 83 + 뷰 1 | 63 | 과도한 세분화 통합, 요약 테이블 정리 |
| 접두어 | RI_/SV_/CO_/RF_/AL_ | INP_/SMR_/OUT_/REF_/AUD_ | 직관성 향상 |
| JSON 보관 | RI_S_RAW_DATA 1개 | INP_RAW_DATASET (동일 역할) | 명칭 변경 |
| 요약 분리 | SV_* 4개 요약 + 4개 검증 | SMR_* 7개 (요약+적격+검증 통합) | 중간 요약과 입력 분리 명확화 |
| 출력 JSON | CO_S_REPORT_JSON | OUT_REPORT_JSON (8섹션 JSONB) | JSON 변환 목적 명확화 |
| 기준정보 | RF_* 26개 | REF_* 26개 (동일) | 명칭만 변경, 구조 유지 |
| 세목 분기 | RI_C_*/RI_I_* | INP_C_*/INP_I_* | 일관된 접두어 체계 |

### 11.3 사용자 요구사항 매핑

| 사용자 요구 | 대응 테이블 | 설계 방식 |
|------------|-----------|----------|
| "세부 입력자료를 JSON 형태의 데이터셋으로 받아서 보관" | **INP_RAW_DATASET** | category별 JSONB 원본 INSERT ONLY |
| "출력은 JSON 형태의 데이터셋으로 변환하기 위한 데이터 보관" | **OUT_REPORT_JSON** (+ OUT_* 정규 테이블) | 정규 테이블 → 8섹션 JSONB 직렬화 |
| "기준정보를 활용하여 환급액 계산에 활용하도록 중간단계 요약" | **SMR_*** (7개 테이블) | INP_* + REF_* 결합하여 계산 가능 형태로 요약 |
| "세부 입력자료와 나누어 설계" | INP_(Layer 1) ≠ SMR_(Layer 2) | 물리적 계층 분리, 접두어 구분 |

---

## 버전 이력

| 버전 | 일자 | 변경 내용 | 작성자 |
|-----|------|---------|--------|
| 0.1 | 2026-02-18 | 초안 — 용어사전, 5계층 63개 테이블 스키마, JSON 입출력 구조 | Phase 1 (PDCA) |
