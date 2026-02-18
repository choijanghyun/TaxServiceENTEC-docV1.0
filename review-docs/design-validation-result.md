# Tax-refund 설계 문서 검증 결과 보고서

> **검증 대상**: Tax-refund.design.md (설계서 v1.2 — Warning 9건 보완 반영)
> **검증일**: 2026-02-18
> **최종 갱신**: 2026-02-18 (v1.2 보완 반영)
> **검증자**: Design Validation Agent
> **완전성 점수**: 93 / 100 (v1.0: 78 → v1.1: 84 → v1.2: 93, Critical 0건)

---

## 목차

1. [검증 항목 1: 계획서 대비 설계서 커버리지](#1-계획서plan-대비-설계서design-커버리지)
2. [검증 항목 2: 스키마 대비 설계서 정합성](#2-스키마schemamd-대비-설계서-정합성)
3. [검증 항목 3: 설계서 내부 일관성](#3-설계서-내부-일관성)
4. [검증 항목 4: 참고1번 대비 누락사항](#4-참고1번-대비-누락사항)
5. [검증 항목 5: 용어/명명 일관성](#5-용어명명-일관성)
6. [종합 이슈 요약](#6-종합-이슈-요약)
7. [권고사항](#7-권고사항)

---

## 1. 계획서(Plan) 대비 설계서(Design) 커버리지

> Plan 문서: `docs/01-plan/features/Tax-refund.plan.md`
> Design 문서: `docs/02-design/features/Tax-refund.design.md`

### 1.1 기능 요구사항 매핑 (FR-01 ~ FR-19)

| FR ID | 요구사항 | Design 반영 여부 | 상태 | 상세 |
|-------|---------|:---------------:|:----:|------|
| FR-01 | req_id 발급 + 요청 접수 (Idempotency-Key) | O | [PASS] | API-01, M1-01, Idempotency-Key 모두 설계 완료 |
| FR-02 | 법인 세무자료 입력 (32개 카테고리) | O | [PASS] | CategoryCode enum 32개, InpRawDataset 엔티티 설계 완료 |
| FR-03 | 개인사업자 세무자료 입력 | O | [PASS] | P1-01~P1-08 개인 전용 검증 모듈 설계 완료 |
| FR-04 | 입력 데이터 107개 규칙 검증 | O | [PASS] | MX-03 ValidationEngine, 107개 규칙 코드 분류 표 포함 |
| FR-05 | 경정청구 불가사유 Hard Fail 차단 | O | [PASS] | M3-00 HardFailCheckService, HardFailException 설계 완료 |
| FR-06 | 중소기업 해당 여부 자동 판정 | O | [PASS] | M3-02 SmeEligibilityService 설계 완료 |
| FR-07 | 소재지 구분 (CORP 본점간주 / INC 사업장별) | O | [PASS] | M3-03 LocationCheckService + P3-01 IncLocationCheckService |
| FR-08 | 상시근로자 수 산정 | O | [PASS] | M3-04 EmployeeCountService, 절사 공식 명시 |
| FR-09 | 통합투자세액공제 SS24 | O | [PASS] | M4-01 InvestmentCreditCalc, F12/F13 공식 |
| FR-10 | 통합고용세액공제 SS29의8 | O | [PASS] | M4-02 EmploymentCreditCalc, F10/F11 공식 |
| FR-11 | 창업중소기업 세액감면 SS6 | O | [PASS] | M4-04 StartupExemptionCalc, F14 공식 |
| FR-12 | 중소기업 특별세액감면 SS7 | O | [PASS] | M4-05 SmeSpecialExemptionCalc, F15/F16 공식 |
| FR-13 | R&D 세액공제 SS10 | O | [PASS] | M4-06 RdCreditCalc, F17/F18 공식 |
| FR-14 | 상호배제 10규칙 자동 적용 | O | [PASS] | M5-01 MutualExclusionService, 10규칙 표 명시 |
| FR-15 | 최적 조합 탐색 (B&B / Greedy) | O | [PASS] | M5-02 CombinationSearchService, 의사코드 포함 |
| FR-16 | 최저한세 자동 적용 | O | [PASS] | M5-03 MinimumTaxService, F04/F-INC-06 공식 |
| FR-17 | 환급액 비교 보고서 8섹션 생성 | O | [PASS] | M6-01~M6-05, OUT_REPORT_JSON 8섹션 구조 |
| FR-18 | RBAC 인증/권한 (JWT, 4단계) | O | [PASS] | SecurityConfig, AccessControlService, 4단계 역할 명시 |
| FR-19 | 세율/공제율 기준정보 연도별 관리 | O | [PASS] | REF_* 26개 테이블, 2018~2025 세율 |

**커버리지 결과**: 19/19 (100%) -- [PASS]

### 1.2 비기능 요구사항 매핑

| NFR | 기준 | Design 반영 | 상태 |
|-----|------|:-----------:|:----:|
| 성능 | 77개 항목 30초 이내 | O (app.calculation.timeout-ms: 30000) | [PASS] |
| 정확성 | 금액 10원 미만 TRUNCATE | O (TruncationUtil, MoneyAmount, TaxConstants) | [PASS] |
| 보안 | JWT 8시간, RBAC 4단계, AES-256 | O (SecurityConfig, CryptoUtil, 보안 설계 섹션) | [PASS] |
| 가용성 | 99.5%, TX ACID | O (TX-1/TX-2 ACID, **18절 SLA 99.5% 설계 추가**) | **[PASS]** ✅ |
| 확장성 | Strategy 패턴 | O (CreditCalculator, CreditCalculatorRegistry) | [PASS] |
| 유지보수 | 80%+ 커버리지, OpenAPI 3.0 | O (Swagger 설정, 테스트 구조) | [PASS] |

### 1.3 Plan에서 명시되었으나 설계서에서 약한 항목

| 항목 | Plan 명시 | Design 반영 수준 | 상태 |
|------|---------|:------------:|:----:|
| ~~사회보험료 §30의4 (Phase 2)~~ | Phase 2로 명시적 이동 | M4-03에 **[Phase 2 — 2024년 일몰]** 명시 | **[PASS]** ✅ |
| ~~E2E 시나리오 10개~~ | Plan에 "법인5+개인5" 명시 | **17절 E2E 10개 시나리오 상세 추가** | **[PASS]** ✅ |
| SonarQube Lint 통합 | Plan NFR에 명시 | 17절 CI/CD 파이프라인에 SonarQube 단계 포함 | [WARN] (상세 설정 미포함) |
| ~~동시 100 요청 처리~~ | Plan 성공 기준 | **17절 JMeter 부하테스트 설계 추가** | **[PASS]** ✅ |

---

## 2. 스키마(schema.md) 대비 설계서 정합성

> Schema 문서: `docs/01-plan/schema.md` (63개 테이블, 5 레이어)
> Design 문서의 DB 관련 섹션

### 2.1 테이블 수/구조 정합성

| 항목 | schema.md | design.md | 상태 | 상세 |
|------|----------|----------|:----:|------|
| 총 테이블 수 | **63개** | **63개** | [PASS] | 12.2절에서 63개 명시 |
| Layer 1 (INP_) | 20개 | 20개 | [PASS] | 일치 |
| Layer 2 (SMR_) | 7개 | 7개 | [PASS] | 일치 |
| Layer 3 (OUT_) | 9개 | 9개 | [PASS] | 일치 |
| Layer 4 (REF_) | 26개 | 26개 | [PASS] | 일치 |
| Layer 5 (AUD_) | 1개 | 1개 | [PASS] | 일치 |

### 2.2 접두어 체계 정합성

| 항목 | schema.md 접두어 | design.md 접두어 | 상태 |
|------|----------------|-----------------|:----:|
| 입력 계층 | INP_ | INP_ | [PASS] |
| 요약 계층 | SMR_ | SMR_ | [PASS] |
| 출력 계층 | OUT_ | OUT_ | [PASS] |
| 기준정보 | REF_ | REF_ | [PASS] |
| 감사 로그 | AUD_ | AUD_ | [PASS] |

### 2.3 세목 분기 접미어 정합성

| 항목 | schema.md | design.md | 상태 |
|------|----------|----------|:----:|
| 법인 전용 | `_C_` (예: INP_C_CORP_BASIC) | INP_C_*, REF_C_* | [PASS] |
| 개인 전용 | `_I_` (예: INP_I_INC_BASIC) | INP_I_*, REF_I_* | [PASS] |
| 공통 | `_S_` (예: INP_S_EMPLOYEE_DETAIL) | INP_S_*, REF_S_* | [PASS] |

### 2.4 핵심 테이블 컬럼 정합성 검증

| 테이블 | schema.md 컬럼 | design.md 반영 | 상태 | 상세 |
|--------|--------------|:-------------:|:----:|------|
| INP_REQUEST | req_id, applicant_id, taxpayer_id, tax_type, tax_year, request_status, idempotency_key 등 15컬럼 | InpRequest.java 엔티티에 주요 컬럼 매핑 | [PASS] | |
| INP_RAW_DATASET | req_id, category, raw_json, schema_version, checksum 등 7컬럼 | InpRawDataset.java 엔티티에 매핑 | [PASS] | |
| SMR_BASIC_INFO | req_id, tax_type, corp_size 등 20컬럼 | SmrBasicInfo 엔티티 언급, 컬럼 상세는 schema.md 참조 위임 | [WARN] | 설계서에서 엔티티 전체 컬럼 정의 생략 |
| OUT_CREDIT_DETAIL | 17컬럼 상세 정의 | OutCreditDetail 엔티티 언급, schema.md 참조 위임 | [WARN] | 동일 |
| AUD_CALCULATION_LOG | 11컬럼 상세 정의 | AudCalculationLog 엔티티 언급, schema.md 참조 위임 | [PASS] | |

### 2.5 schema.md와 design.md의 구조적 불일치

| 이슈 | 상세 | 상태 |
|------|------|:----:|
| ~~SMR_VALIDATION_LOG~~ | ~~JPA 엔티티 디렉토리에 누락~~ | **RESOLVED** ✅ v1.2: SmrValidationLog.java 추가 | **[PASS]** ✅ |
| ~~SMR_PREP 테이블~~ | ~~entity/smr/ 하위에 미명시~~ | **RESOLVED** ✅ v1.2: SmrPrep.java 추가 | **[PASS]** ✅ |
| ~~SMR_ELIGIBILITY 테이블~~ | ~~엔티티 클래스 디렉토리 누락~~ | **RESOLVED** ✅ v1.2: SmrEligibility.java 추가 | **[PASS]** ✅ |

---

## 3. 설계서 내부 일관성

### 3.1 Java 버전 불일치 ~~(Critical)~~ → **RESOLVED (v1.1)**

| 위치 | v1.0 (수정 전) | v1.1 (수정 후) | 상태 |
|------|:-------------:|:-------------:|:----:|
| Plan 4.2 핵심 아키텍처 결정 | **Java 17** | **Java 17** | 기준 |
| Design 1.1 핵심 기술 스택 | ~~Java 1.8 이상 (8+)~~ | **Java 17 이상 (17+)** | [PASS] ✅ |
| Design 1.2 개발 환경 JDK | ~~Oracle JDK 1.8 / OpenJDK 8+~~ | **Oracle JDK 17 / OpenJDK 17+** | [PASS] ✅ |
| Design 3. pom.xml `<java.version>` | ~~1.8~~ | **17** | [PASS] ✅ |
| 참고1번 2.1 기술 스택 | **Java 17** | **Java 17** | 기준 |

**분석**: ~~Plan과 참고1번 모두 Java 17을 명시하고 있으나, 설계서는 Java 1.8(8)로 기술하고 있었다.~~ **v1.1에서 3곳 모두 Java 17로 수정 완료.** Plan, 참고1번, 설계서 간 Java 버전이 통일되었다.

### 3.2 API 인증 방식 불일치

| 위치 | 인증 방식 | 상태 |
|------|---------|:----:|
| Plan 4.2 | Spring Security + JWT | 기준 |
| Design 1.1, 5.1, 9.1, 15.1 | Spring Security + JWT, RBAC 4단계 | [PASS] |
| Design 13.2 API-01 | `Authorization: Bearer {JWT}` | [PASS] |
| 참고1번 4.1 기본 사항 | **API Key (X-API-Key 헤더)** | 참고 |
| 참고1번 10.1 보안 요건 | API Key (X-API-Key 헤더) | 참고 |

**분석**: 설계서(design.md)는 JWT 방식으로 통일되어 있어 Plan과 일치한다. 참고1번은 API Key 방식이었으나 설계서에서 JWT로 개선한 것은 적절하다. 다만, 설계서 내에서 API Key 방식에 대한 언급이 없으므로, 참고1번과의 차이가 의도적인지 명시적 기록이 필요하다.

### 3.3 테이블 접두어 참조 혼재 (design.md 내부)

설계서 내부에서 참고1번의 옛 접두어와 현 schema.md의 접두어가 혼재된 부분:

| 위치 | 사용된 접두어 | 올바른 접두어 | 상태 |
|------|------------|:----------:|:----:|
| design.md 전반 | INP_, SMR_, OUT_, REF_, AUD_ | INP_, SMR_, OUT_, REF_, AUD_ | [PASS] |

**분석**: 설계서 내부에서는 접두어가 일관적으로 schema.md와 일치한다. 참고1번의 RI_/SV_/CO_/RF_/AL_ 접두어는 설계서에서 사용되지 않는다. [PASS]

### 3.4 테스트 공식 수 ~~불일치~~ → **통일 완료 (v1.1/v1.2)**

| 위치 | 단위 테스트 대상 공식 수 | 상태 |
|------|:-------------------:|:----:|
| Plan 6.1 (v1.1) | **56개** 계산 공식 단위 테스트 100% | **[PASS]** ✅ |
| Design 1.1 (v1.2) | **56개** 공식 단위 테스트 필수 | **[PASS]** ✅ |
| Design 11.3 (v1.2) | **56개** 전체 공식 목록 (F01~F41, F-INC-01~11, F-COM-01) | **[PASS]** ✅ |
| 참고1번 2.1 기술 스택 | **56개** 공식 단위 테스트 | [PASS] |
| 참고1번 12.1 테스트 범위 | **56개** 계산 공식 67건+ | [PASS] |

**분석**: v1.1(Plan)과 v1.2(Design)에서 모두 56개로 통일됨. F11(통합고용 추가), F-COM-01(사후관리 추징), F-INC-07~11(개인 공식 5개)를 포함한 정확한 총 수가 3개 문서 모두에서 일치한다.

### 3.5 TX-1 범위 불일치

| 위치 | TX-1 범위 | 상태 |
|------|---------|:----:|
| 참고1번 9.1 트랜잭션 경계 | M1-01 ~ M1-03 | - |
| Design 11.1 계산 파이프라인 | M1-01 ~ M1-12 / P1-01~08 | [PASS] |
| Design 14.1 트랜잭션 경계 | M1-01 ~ M1-12 / P1-01~08 | [PASS] |

**분석**: 설계서는 v2.1에서 추가된 M1-04~12, P1-01~08을 TX-1 범위에 포함하여 정합적이다. 참고1번의 M1-01~M1-03 범위는 v2.0 이전 기준이므로 설계서의 확장이 적절하다.

### 3.6 API 엔드포인트 수 정합성 → **통일 완료 (v1.1)**

| 위치 | API 수 | 비고 |
|------|:------:|------|
| Plan 4.3 (v1.1) | **11개** 엔드포인트 (API-01~11) | **[PASS]** ✅ v1.1에서 7→11개 수정 |
| Design 13.1 | 11개 엔드포인트 (API-01~11) | [PASS] |
| 참고1번 4.2 | 11개 엔드포인트 (API-01~11) | [PASS] |

**분석**: Plan v1.1에서 API 수를 11개로 수정하여 3개 문서 모두 일치. **[PASS]** ✅

### 3.7 application/port 디렉토리 → **추가 완료 (v1.2)**

| 위치 | 내용 | 상태 |
|------|------|:----:|
| Design 2. 디렉토리 구조 (v1.2) | `application/port/in/` + `application/port/out/` **추가** | **[PASS]** ✅ |
| 참고1번 2.3 패키지 구조 | `application/port/in/`, `application/port/out/` 포함 | 일치 |

**분석**: v1.2에서 인바운드 포트(CreateRequestUseCase, AnalyzeUseCase, ReportUseCase, ReferenceDataUseCase)와 아웃바운드 포트(PersistRequestPort, LoadSummaryPort, PersistCreditResultPort, LoadReferenceDataPort)를 설계서 디렉토리 구조에 추가. **[PASS]** ✅

---

## 4. 참고1번 대비 누락사항

> 참고1번: `참고1번-development-design-v2.md` (v2.1)
> Design 문서: `docs/02-design/features/Tax-refund.design.md`

### 4.1 참고1번에 있으나 설계서에 누락된 핵심 요소

| 항목 | 참고1번 위치 | 설계서 반영 | 상태 | 심각도 |
|------|-----------|:---------:|:----:|:-----:|
| M2 기준정보 모듈 | 참고1번 2.3 패키지: `m2/` (기준정보) | 설계서에 m2/ 패키지 대신 ReferenceDataService + port/out으로 대체 | [WARN] | Low |
| ~~**RF_* 갱신 프로세스**~~ | ~~참고1번 10.3 외부 참조 데이터 갱신~~ | **v1.2: 12.4절 갱신 프로세스 추가** (9개 테이블, 5단계 절차, 검증 체크리스트) | **[PASS]** ✅ | ~~High~~ |
| RF_* 자동 배치 스케줄 | 참고1번 10.3 (환율 매일, 인정이자율 매월 등) | v1.2: 12.4절에 갱신 주기 표 포함 | **[PASS]** ✅ | ~~Medium~~ |
| M4-10 결손금 소급공제 NPV | 참고1번 13.5 의사코드 | M4-08 + P4-09 존재, M4-10 파일명 미명시 | [WARN] | Low |
| 보고서 3종 상세 구조 | 참고1번 8.2 경영진용/세무대리인용/국세청용 | M6-04만 존재, 3종 구분 상세 없음 | [WARN] | Low |
| ~~**F-COM-01 사후관리 추징액**~~ | ~~참고1번 6.3 상세 의사코드~~ | **v1.2: 11.3절에 F-COM-01 공식+의사코드 추가** | **[PASS]** ✅ | ~~Medium~~ |
| ~~**F11 통합고용 추가공제**~~ | ~~참고1번 6.1 F11 상세~~ | **v1.2: 11.3절에 F11 경력단절/정규직/육아복귀 추가** | **[PASS]** ✅ | ~~Medium~~ |
| ~~**최저한세율 구간 상세**~~ | ~~참고1번 6.5 구간별~~ | **v1.2: 11.3절에 구간별 7~17% 표 + R&D 3단계 추가** | **[PASS]** ✅ | ~~Medium~~ |
| **v1.0 vs v2.0 변경 이력표** | 참고1번 1.5 변경사항 상세 매트릭스 | 설계서에 없음 (v1.0이 초판이므로 불필요) | [PASS] | - |

### 4.2 참고1번에서 의도적으로 변경된 항목 (적절한 변경)

| 항목 | 참고1번 | 설계서 | 판단 |
|------|-------|-------|:----:|
| 테이블 수 | 83개 + 뷰 1 | 63개 | [PASS] -- schema.md에 근거한 간소화, 변경 이유 명시 |
| 접두어 | RI_/SV_/CO_/RF_/AL_ | INP_/SMR_/OUT_/REF_/AUD_ | [PASS] -- 직관성 향상, schema.md와 일치 |
| API 인증 | API Key (X-API-Key) | JWT (Authorization: Bearer) | [PASS] -- Plan 방향과 일치, 보안 강화 |
| 기준정보 테이블 접두어 | RF_ | REF_ | [PASS] -- schema.md와 일치 |

### 4.3 참고1번의 계산 공식 커버리지 → **전체 포함 (v1.2)**

| 공식 그룹 | 참고1번 | 설계서 v1.2 반영 | 상태 |
|----------|-------|:---------:|:----:|
| F01~F04 (과세표준/산출세액/최저한세) | O | O (F01~F04 전체) | **[PASS]** ✅ |
| F10~F13 (통합고용/투자) | O | O (F10~F13 전체) | **[PASS]** ✅ |
| F14~F19 (창업/중소특별/R&D/사보) | O | O (F14~F19 전체) | **[PASS]** ✅ |
| F30~F34 (환급액/가산금) | O | O (F30~F34 전체) | **[PASS]** ✅ |
| F40~F41 (농특세) | O | **O (v1.2 추가)** | **[PASS]** ✅ |
| F-INC-01~F-INC-11 | O | **O (v1.2: 전체 11개 포함)** | **[PASS]** ✅ |
| F-COM-01 (사후관리 추징) | O | **O (v1.2 추가)** | **[PASS]** ✅ |

**분석**: v1.2에서 11.3절을 전면 확장하여 56개 공식 전체(F01~F41 + F-INC-01~11 + F-COM-01)를 8개 서브섹션으로 체계적으로 나열. 참고1번과의 공식 커버리지가 100%에 도달했다.

---

## 5. 용어/명명 일관성

### 5.1 테이블명 일관성

| 구분 | schema.md 명칭 | design.md 명칭 | 참고1번 명칭 | 상태 |
|------|--------------|--------------|------------|:----:|
| 요청 마스터 | INP_REQUEST | INP_REQUEST (InpRequest.java) | RI_S_REQUEST | [PASS] -- schema/design 일치 |
| JSON 원본 | INP_RAW_DATASET | INP_RAW_DATASET (InpRawDataset.java) | RI_S_RAW_DATA | [PASS] |
| 기초정보 요약 | SMR_BASIC_INFO | SMR_BASIC_INFO (SmrBasicInfo.java) | SV_S_BASIC | [PASS] |
| 고용 요약 | SMR_EMPLOYEE | SMR_EMPLOYEE (SmrEmployee.java) | SV_S_EMPLOYEE | [PASS] |
| 산출 결과 | OUT_CREDIT_DETAIL | OUT_CREDIT_DETAIL (OutCreditDetail.java) | CO_S_CREDIT_DETAIL | [PASS] |
| 보고서 JSON | OUT_REPORT_JSON | OUT_REPORT_JSON (OutReportJson.java) | CO_S_REPORT_JSON | [PASS] |

### 5.2 엔티티-테이블 명명 규칙 일관성

| 테이블 | 엔티티 클래스명 | 명명 규칙 | 상태 |
|--------|-------------|---------|:----:|
| inp_request | InpRequest | PascalCase, 접두어 유지 | [PASS] |
| inp_raw_dataset | InpRawDataset | PascalCase, 접두어 유지 | [PASS] |
| smr_basic_info | SmrBasicInfo | PascalCase, 접두어 유지 | [PASS] |
| smr_employee | SmrEmployee | PascalCase, 접두어 유지 | [PASS] |
| smr_deduction_item | SmrDeductionItem | PascalCase, 접두어 유지 | [PASS] |
| smr_financial | SmrFinancial | PascalCase, 접두어 유지 | [PASS] |
| out_credit_detail | OutCreditDetail | PascalCase, 접두어 유지 | [PASS] |
| out_combination | OutCombination | PascalCase, 접두어 유지 | [PASS] |
| out_refund | OutRefund | PascalCase, 접두어 유지 | [PASS] |
| out_report_json | OutReportJson | PascalCase, 접두어 유지 | [PASS] |
| aud_calculation_log | AudCalculationLog | PascalCase, 접두어 유지 | [PASS] |
| ref_tax_rate_bracket | RefTaxRateBracket | PascalCase, 접두어 유지 | [PASS] |

**규칙**: `{테이블접두어}_{테이블명}` -> `{PascalCase접두어}{PascalCase테이블명}` -- 전체 일관 [PASS]

### 5.3 조항 코드 표기 일관성

| 조항 | schema.md | design.md | 참고1번 | 상태 |
|------|----------|----------|--------|:----:|
| 통합투자 | SS24 | SS24 (TaxConstants: "SS24") | SS24 | [PASS] |
| 통합고용 | SS29_8 | SS29_8 (TaxConstants: "SS29_8") | SS29의8 | [WARN] |
| 사회보험 | SS30_4 | SS30_4 (TaxConstants: "SS30_4") | SS30의4 | [WARN] |
| 창업감면 | SS6 | SS6 | SS6 | [PASS] |
| 중소특별 | SS7 | SS7 | SS7 | [PASS] |
| R&D | SS10 | SS10 | SS10 | [PASS] |

**분석**: "SS29의8" vs "SS29_8", "SS30의4" vs "SS30_4" -- 참고1번에서는 한글 "의"를 사용하고, design/schema에서는 언더스코어 "_"를 사용한다. 코드 내에서는 언더스코어가 적절하나, 문서 텍스트 서술 부분에서 혼용이 있을 수 있다. design.md 내부에서는 일관적이므로 [PASS]로 최종 판단.

### 5.4 패키지 구조 내 모듈 코드 일관성

| 모듈 | 디렉토리 | 파일명 | 주석/코드 내 코드 | 상태 |
|------|---------|--------|:------------:|:----:|
| M1-01 | m1/ | RequestCreationService.java | M1-01 | [PASS] |
| M3-00 | m3/ | HardFailCheckService.java | M3-00 | [PASS] |
| M4-01 | m4/ | InvestmentCreditCalc.java | M4-01 SS24 | [PASS] |
| P1-01 | p1/ | IncBasicValidator.java | P1-01 | [PASS] |
| P4-01 | p4/ | IncomeDeductionCalc.java | P4-01 | [PASS] |
| MX-01 | mx/ | CircularReferenceResolver.java | MX-01 | [PASS] |

### 5.5 Enum 명명 일관성

| Enum | design.md 값 | schema.md 값 | 상태 |
|------|------------|------------|:----:|
| TaxType | CORP / INC | CORP / INC | [PASS] |
| CorpSize | SMALL / MEDIUM / MID_LARGE / LARGE | SMALL / MEDIUM / MID_LARGE / LARGE | [PASS] |
| CapitalZone | OVER_CONCENTRATION / GROWTH_MGMT / NON_CAPITAL / DEPOPULATION | 동일 | [PASS] |
| RequestStatus | received ~ completed / error (10단계) | 동일 | [PASS] |
| CreditType | EXEMPTION / CREDIT / DEDUCTION | EXEMPTION / CREDIT / DEDUCTION | [PASS] |

### 5.6 용어 사용 규칙 준수

| 규칙 (schema.md 1.4) | 설계서 준수 여부 | 상태 |
|---------------------|:----------:|:----:|
| 코드에서 영문: camelCase | O (reqId, taxType, calculatedTax) | [PASS] |
| DB 컬럼: snake_case | O (req_id, tax_type, calculated_tax) | [PASS] |
| API 응답: camelCase | O (reqId, datasetsReceived, createdAt) | [PASS] |
| 세목 코드: CORP/INC | O (전체 문서 일관) | [PASS] |

---

## 6. 종합 이슈 요약

### 6.1 Critical (구현 불가능) -- ~~1건~~ **0건 (v1.1 수정 완료)**

| No | 이슈 | 상태 | 수정 내용 |
|----|------|:----:|---------|
| ~~C-01~~ | ~~Java 버전 불일치~~ | **RESOLVED** ✅ | v1.1에서 Java 17로 3곳 수정 완료 (1.1절, 1.2절, pom.xml) |

### 6.2 Warning (개선 필요) -- ~~15건~~ → **6건 (v1.2에서 9건 해소)**

| No | 이슈 | 심각도 | 위치 | 상태 |
|----|------|:------:|------|:----:|
| ~~W-01~~ | ~~SMR_ELIGIBILITY, SMR_PREP 엔티티 클래스 누락~~ | ~~High~~ | ~~design.md 2.~~ | **RESOLVED** ✅ |
| ~~W-02~~ | ~~RF_* 기준정보 갱신 프로세스 미설계~~ | ~~High~~ | ~~design.md~~ | **RESOLVED** ✅ |
| ~~W-03~~ | ~~계산 공식 전체 목록 미포함 (12개만 나열)~~ | ~~High~~ | ~~design.md 11.3~~ | **RESOLVED** ✅ |
| ~~W-04~~ | ~~농특세 공식 (F40/F41) 미포함~~ | ~~Medium~~ | ~~design.md 11.3~~ | **RESOLVED** ✅ |
| ~~W-05~~ | ~~F-COM-01 사후관리 추징액 공식 미포함~~ | ~~Medium~~ | ~~design.md 11.3~~ | **RESOLVED** ✅ |
| ~~W-06~~ | ~~M4-03 Phase 2 미표시~~ | ~~Medium~~ | ~~design.md 2.~~ | **RESOLVED** ✅ |
| ~~W-07~~ | ~~application/port 패키지 설계 누락~~ | ~~Medium~~ | ~~design.md 2.~~ | **RESOLVED** ✅ |
| ~~W-08~~ | ~~부하 테스트 방법/도구 미설계~~ | ~~Medium~~ | ~~design.md~~ | **RESOLVED** ✅ |
| ~~W-09~~ | ~~E2E 테스트 시나리오 10개 미정의~~ | ~~Medium~~ | ~~design.md~~ | **RESOLVED** ✅ |
| W-10 | SonarQube CI/CD 파이프라인 통합 설계 미포함 | Low | design.md 전체 | Open |
| ~~W-11~~ | ~~99.5% 가용성 SLA 미설계~~ | ~~Medium~~ | ~~design.md~~ | **RESOLVED** ✅ (18절 추가) |
| W-12 | M2 기준정보 모듈 패키지(m2/)가 참고1번에 명시되었으나 설계서에는 없음 | Low | design.md 2. 디렉토리 | Open |
| W-13 | 보고서 3종(경영진용/세무대리인용/국세청용) 상세 구조 미포함 | Low | design.md 전체 | Open |
| ~~W-14~~ | ~~최저한세율 구간별 상세 + R&D 배제 3단계 미포함~~ | ~~Medium~~ | ~~design.md 11.3~~ | **RESOLVED** ✅ |
| ~~W-15~~ | ~~Plan API 수 불일치 (7개→11개)~~ | ~~Low~~ | ~~Plan 4.3~~ | **RESOLVED** ✅ (Plan v1.1 수정) |

### 6.3 Info (참고) -- 3건

| No | 이슈 | 위치 |
|----|------|------|
| I-01 | 참고1번의 접두어(RI_/SV_/CO_/RF_/AL_)가 의도적으로 INP_/SMR_/OUT_/REF_/AUD_로 변경됨. schema.md 11.2에 변경 이유 명시 | schema.md 11.2 |
| I-02 | 테이블 수가 83개(참고1번) -> 63개(schema.md/설계서)로 간소화됨. schema.md 11.2에 "과도한 세분화 통합" 변경 이유 명시 | schema.md 11.2 |
| I-03 | API 인증이 API Key(참고1번) -> JWT(설계서)로 변경됨. Plan 방향과 일치하는 적절한 변경 | design.md 15.1 |

---

## 7. 권고사항

### 7.1 즉시 수정 필요 (구현 착수 전 필수)

| 순위 | 권고 | 대상 파일 | 예상 공수 | 상태 |
|:----:|------|---------|:--------:|:----:|
| ~~1~~ | ~~Java 버전을 17로 통일~~ | ~~design.md~~ | ~~10분~~ | **RESOLVED** ✅ |
| ~~2~~ | ~~SMR_ELIGIBILITY, SMR_PREP 엔티티 추가~~ | ~~design.md~~ | ~~20분~~ | **RESOLVED** ✅ v1.2 |
| ~~3~~ | ~~RF_* 갱신 프로세스 섹션 추가~~ | ~~design.md~~ | ~~1시간~~ | **RESOLVED** ✅ v1.2 (12.4절) |

### 7.2 우선 보완 권고 (구현 초기에 보완)

| 순위 | 권고 | 대상 파일 | 예상 공수 |
|:----:|------|---------|:--------:|
| ~~4~~ | ~~계산 공식 전체 목록~~ | ~~design.md 11.3~~ | ~~2시간~~ | **RESOLVED** ✅ v1.2 (56개 전체) |
| ~~5~~ | ~~농특세/최저한세 상세~~ | ~~design.md 11.3~~ | ~~1시간~~ | **RESOLVED** ✅ v1.2 |
| ~~6~~ | ~~M4-03 Phase 2 명시~~ | ~~design.md 2.~~ | ~~10분~~ | **RESOLVED** ✅ v1.2 |
| ~~7~~ | ~~테스트 계획 섹션~~ | ~~design.md 17절~~ | ~~2시간~~ | **RESOLVED** ✅ v1.2 |

### 7.3 향후 보완 권고 (구현 진행 중 보완 가능)

| 순위 | 권고 | 대상 파일 |
|:----:|------|---------|
| ~~8~~ | ~~application/port 패키지 설계 추가~~ | ~~design.md 2.~~ | **RESOLVED** ✅ v1.2 |
| ~~9~~ | ~~가용성 SLA 99.5% 설계~~ | ~~design.md 18절~~ | **RESOLVED** ✅ v1.2 |
| 10 | 보고서 3종(경영진/세무대리인/국세청) 상세 구조 추가 | design.md | Open (Low) |
| ~~11~~ | ~~Plan API 수 7→11개~~ | ~~Plan 문서~~ | **RESOLVED** ✅ Plan v1.1 |
| ~~12~~ | ~~공식 수 53→56개~~ | ~~design.md, Plan~~ | **RESOLVED** ✅ v1.1/v1.2 |

---

## 체크리스트 결과 요약

| 검증 항목 | 결과 | 비고 |
|----------|:----:|------|
| Plan 대비 기능 요구사항 커버리지 (FR-01~19) | PASS (19/19) | 100% 매핑 |
| Plan 대비 비기능 요구사항 커버리지 | **PASS** ✅ | v1.2: SLA 99.5%, JMeter 부하테스트 설계 추가 |
| Schema 테이블 수/구조 정합성 | PASS | 63개, 5 레이어 일치 |
| Schema 접두어/세목 분기 정합성 | PASS | INP_/SMR_/OUT_/REF_/AUD_, _C_/_I_/_S_ 일치 |
| Schema 컬럼 레벨 정합성 | WARN | 일부 엔티티 전체 컬럼 미기재 (schema 참조 위임) |
| Schema 엔티티 클래스 누락 | **PASS** ✅ | v1.2: SmrEligibility, SmrPrep, SmrValidationLog 추가 |
| **내부 Java 버전 일관성** | **PASS** ✅ | **v1.1에서 Java 17로 통일 완료** |
| 내부 API 인증 방식 일관성 | PASS | JWT 통일 |
| 내부 접두어 일관성 | PASS | INP_/SMR_/OUT_/REF_/AUD_ 통일 |
| 내부 TX 범위 일관성 | PASS | TX-1 (M1~P1), TX-2 (M3~M6) 일관 |
| 참고1번 대비 RF_* 갱신 프로세스 | **PASS** ✅ | v1.2: 12.4절 갱신 프로세스 추가 |
| 참고1번 대비 계산 공식 완전성 | **PASS** ✅ | v1.2: 56개 전체 포함 (F01~F41, F-INC-01~11, F-COM-01) |
| 용어 일관성 | PASS | CORP/INC, camelCase/snake_case 규칙 준수 |
| 엔티티/테이블 명명 일관성 | PASS | PascalCase 변환 규칙 일관 |
| 조항 코드 일관성 | PASS | SS24, SS29_8 등 코드 내 일관 |

---

## 점수 산출 근거

| 카테고리 | 배점 | v1.1 득점 | v1.2 득점 | v1.2 변동 사유 |
|---------|:----:|:--------:|:--------:|---------|
| Plan 대비 커버리지 | 20 | 18 | **20** | 테스트 계획(E2E 10건, 부하테스트), SLA 설계 추가 (+2) |
| Schema 정합성 | 20 | 17 | **20** | SmrEligibility, SmrPrep, SmrValidationLog 엔티티 추가 (+3) |
| 내부 일관성 | 25 | 23 | **24** | 공식 수 56개 통일, 테스트 수 일치 (+1) |
| 참고1번 대비 완전성 | 20 | 14 | **18** | RF_* 갱신 프로세스 추가 (+3), 공식 56개 전체 포함 (+3), 보고서3종 미포함 (-2) |
| 용어/명명 일관성 | 15 | 12 | **11** | port 패키지 추가 (+2), M2 패키지 미포함 (-1), 보고서 상세 미포함 (-2) |
| **합계** | **100** | **84** | **93** | v1.0: 78 → v1.1: 84 → v1.2: **93** (Warning 9건 해소) |

---

## 최종 판정

**완전성 점수: 93/100** (v1.0: 78 → v1.1: 84 → v1.2: 93)

> 점수 90~100 구간: **구현 착수 가능 (PDCA 기준 통과)**

v1.2에서 Warning 9건이 추가로 해소되었다:
- W-01: SMR_ELIGIBILITY, SMR_PREP, SMR_VALIDATION_LOG 엔티티 3개 디렉토리 구조에 추가 ✅
- W-02: RF_* 기준정보 갱신 프로세스 섹션(12.4절) 추가 — 9개 테이블, 갱신 절차, 검증 체크리스트 ✅
- W-03: 계산 공식 56개 전체 목록(11.3절) 포함 — F01~F41, F-INC-01~11, F-COM-01 ✅
- W-04/W-05: 농특세 F40/F41, F-COM-01 사후관리 추징액 공식 추가 ✅
- W-06: M4-03 SocialInsuranceCreditCalc에 [Phase 2] 명시 ✅
- W-07: application/port 패키지(인바운드/아웃바운드 포트 인터페이스) 설계 추가 ✅
- W-08/W-09: 테스트 계획 섹션(17절) 추가 — E2E 10건 상세, JMeter 부하테스트, SonarQube CI/CD ✅
- W-11: 가용성 및 모니터링 설계 섹션(18절) 추가 — 99.5% SLA, RTO/RPO, 알림 체계 ✅
- W-14: 최저한세율 구간별 상세 표 + R&D 배제 3단계 추가 ✅
- W-15: Plan API 수 7→11개 반영 (Plan v1.1 수정) ✅

잔여 Warning 3건은 Low 심각도 항목으로 구현 착수에 영향 없음:
- W-10: SonarQube CI/CD 파이프라인 상세 설정 (구현 단계에서 보완 가능)
- W-12: M2 기준정보 모듈 패키지 (ReferenceDataService로 대체 운영)
- W-13: 보고서 3종 상세 구조 (구현 단계에서 상세화)

---

## 버전 이력

| 버전 | 일자 | 변경 내용 | 작성자 |
|-----|------|---------|--------|
| 1.0 | 2026-02-18 | 최초 검증 -- 5개 검증 항목 전수 수행 | Design Validation Agent |
| 1.1 | 2026-02-18 | Critical 수정 반영 -- C-01 Java 17 수정(RESOLVED), 상호배제 10규칙 테이블 추가 반영. 점수 78→84 | Design Validation Agent |
| 1.2 | 2026-02-18 | Warning 9건 보완 반영 -- SMR 엔티티 3개, RF_* 갱신 프로세스, 공식 56개 전체, 농특세/최저한세 상세, F-COM-01, port 패키지, 테스트 계획, SLA 설계, M4-03 Phase 2 명시. 점수 84→93 | Design Validation Agent |
