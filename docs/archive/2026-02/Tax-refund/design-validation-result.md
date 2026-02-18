> **[ARCHIVE]** 2026-02-18 아카이브. 문서 내 경로는 아카이브 전 구조 기준. 실제 위치: `docs/archive/2026-02/Tax-refund/`

# Tax-refund 설계 문서 검증 결과 보고서

> **검증 대상**: Tax-refund.design.md (설계서 v1.2)
> **검증일**: 2026-02-18
> **최종 갱신**: 2026-02-18 (v1.3 전수 검증 -- FR-20/FR-21 추가, US 매핑, 잔여 수치 불일치 발견)
> **검증자**: Design Validation Agent
> **완전성 점수**: 97 / 100 (v1.0: 78 -> v1.1: 84 -> v1.2: 93 -> v1.3: 91 -> v1.4: 95 -> v1.5: 97, Critical 0건)

---

## 목차

1. [검증 항목 1: 계획서 대비 설계서 커버리지 (FR-01~FR-21)](#1-계획서plan-대비-설계서design-커버리지)
2. [검증 항목 2: 요구사항 대비 커버리지 (US-001~US-027)](#2-요구사항-대비-설계서-커버리지)
3. [검증 항목 3: 스키마 대비 설계서 정합성](#3-스키마schemamd-대비-설계서-정합성)
4. [검증 항목 4: 설계서 내부 일관성](#4-설계서-내부-일관성)
5. [검증 항목 5: 참고1번 대비 누락사항](#5-참고1번-대비-누락사항)
6. [검증 항목 6: 용어/명명 일관성](#6-용어명명-일관성)
7. [종합 이슈 요약](#7-종합-이슈-요약)
8. [권고사항](#8-권고사항)

---

## 1. 계획서(Plan) 대비 설계서(Design) 커버리지

> Plan 문서: `docs/01-plan/features/Tax-refund.plan.md` (v1.2, 21개 FR, 177 SP)
> Design 문서: `docs/02-design/features/Tax-refund.design.md` (v1.2)

### 1.1 기능 요구사항 매핑 (FR-01 ~ FR-21)

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
| **FR-20** | **환급가산금 기본 계산 (본세/중간예납 기산일 분리)** | **O** | **[PASS]** | **M6-02 RefundInterestService, F31/F32/F-INC-09 공식, TruncationUtil.truncateInterest()** |
| **FR-21** | **지방소득세 환급액 안내 (본세x10%)** | **O** | **[PASS]** | **M6-03 LocalTaxGuideService, F33 공식, 보고서 8섹션 중 섹션 8에 포함** |

**커버리지 결과**: 21/21 (100%) -- [PASS]

**주의**: Plan v1.1에서 FR-20(환급가산금)과 FR-21(지방소득세)이 추가되었다. 설계서에는 M6-02, M6-03 모듈과 F31~F33, F-INC-09 공식으로 이미 설계되어 있으나, 기존 검증 결과(v1.2)에서는 FR-01~FR-19만 검증하고 FR-20/FR-21을 매핑하지 않았다. 본 v1.3에서 보완.

### 1.2 비기능 요구사항 매핑

| NFR | 기준 | Design 반영 | 상태 |
|-----|------|:-----------:|:----:|
| 성능 | 77개 항목 30초 이내 | O (app.calculation.timeout-ms: 30000) | [PASS] |
| 정확성 | 금액 10원 미만 TRUNCATE | O (TruncationUtil, MoneyAmount, TaxConstants) | [PASS] |
| 보안 | JWT 8시간, RBAC 4단계, AES-256 | O (SecurityConfig, CryptoUtil, 보안 설계 섹션) | [PASS] |
| 가용성 | 99.5%, TX ACID | O (TX-1/TX-2 ACID, 18절 SLA 99.5% 설계) | [PASS] |
| 확장성 | Strategy 패턴 | O (CreditCalculator, CreditCalculatorRegistry) | [PASS] |
| 유지보수 | 80%+ 커버리지, OpenAPI 3.0 | O (Swagger 설정, 테스트 구조) | [PASS] |

### 1.3 Plan에서 명시되었으나 설계서에서 약한 항목

| 항목 | Plan 명시 | Design 반영 수준 | 상태 |
|------|---------|:------------:|:----:|
| ~~사회보험료 SS30의4 (Phase 2)~~ | Phase 2로 명시적 이동 | M4-03에 [Phase 2 -- 2024년 일몰] 명시 | **[PASS]** |
| ~~E2E 시나리오 10개~~ | Plan에 "법인5+개인5" 명시 | 17절 E2E 10개 시나리오 상세 추가 | **[PASS]** |
| SonarQube Lint 통합 | Plan NFR에 명시 | 17절 CI/CD 파이프라인에 SonarQube 단계 포함 | [WARN] (상세 설정 미포함) |
| ~~동시 100 요청 처리~~ | Plan 성공 기준 | 17절 JMeter 부하테스트 설계 추가 | **[PASS]** |

### 1.4 Plan 내부 불일치 발견 (v1.3 신규)

| 위치 | 내용 | 상태 | 심각도 |
|------|------|:----:|:------:|
| Plan 6.1 Definition of Done | "**19개** 기능 요구사항 전수 구현" | **[WARN]** | Medium |
| Plan 3.1 기능 요구사항 표 | "**21개** 기능, 총 177 SP" (FR-20, FR-21 포함) | 기준 | - |

**분석**: Plan v1.1에서 FR-20/FR-21을 3.1절에 추가했으나, 6.1절 Definition of Done의 "19개"를 "21개"로 갱신하지 않았다. 또한 Plan 4.3절 Clean Architecture 도표에 "JPA Entity (83개 테이블)"로 기재되어 있으나, schema.md는 63개 테이블이며 설계서도 63개이다. Plan 자체의 일부 수치가 v1.1 업데이트 시 누락된 것으로 판단된다.

---

## 2. 요구사항 대비 설계서 커버리지

> 요구사항 정의서: `docs/requirements-tax-refund-system.md` (페르소나 5, 에픽 10, 스토리 27)
> Design 문서: `docs/02-design/features/Tax-refund.design.md` (v1.2)

### 2.1 Phase 1 MVP 스토리 매핑 (19개 Must Have + 2개 추가)

| US ID | 스토리 | FR 매핑 | Design 반영 | 상태 |
|:-----:|--------|:------:|:-----------:|:----:|
| US-001 | 경정청구 요청 생성 (req_id) | FR-01 | M1-01, API-01, ReqIdGenerator | [PASS] |
| US-002 | 법인 세무자료 입력 (32개 카테고리) | FR-02 | CategoryCode 32개, InpRawDataset, INSERT ONLY | [PASS] |
| US-003 | 개인사업자 세무자료 입력 | FR-03 | P1-01~P1-08, 다사업장/공동사업 검증 | [PASS] |
| US-004 | 입력 데이터 107개 규칙 검증 | FR-04 | MX-03 ValidationEngine, 6유형 분류 | [PASS] |
| US-005 | 경정청구 불가사유 차단 | FR-05 | M3-00 HardFailCheckService, 3종 차단 | [PASS] |
| US-006 | 중소기업 해당 여부 판정 | FR-06 | M3-02 SmeEligibilityService, CorpSize enum | [PASS] |
| US-007 | 소재지 구분 판단 | FR-07 | M3-03 + P3-01, CapitalZone 4구분 | [PASS] |
| US-008 | 상시근로자 수 산정 | FR-08 | M3-04 EmployeeCountService, 세목별 제외 대상 | [PASS] |
| US-009 | 통합투자세액공제 SS24 | FR-09 | M4-01, F12/F13 공식, 2025 추가율 상향 | [PASS] |
| US-010 | 통합고용세액공제 SS29의8 | FR-10 | M4-02, F10/F11 공식, 이월 10년 | [PASS] |
| US-011 | 창업중소기업 세액감면 SS6 | FR-11 | M4-04, F14 공식, 2025 비수도권 75% | [PASS] |
| US-012 | 중소기업 특별세액감면 SS7 | FR-12 | M4-05, F15/F16, 한도 1억 | [PASS] |
| US-013 | R&D 세액공제 SS10 | FR-13 | M4-06, F17/F18, 3단계 배제 | [PASS] |
| US-017 | 상호배제 10규칙 | FR-14 | M5-01, 10규칙 상세 표 | [PASS] |
| US-018 | 최적 조합 탐색 | FR-15 | M5-02, B&B/Greedy 의사코드 | [PASS] |
| US-019 | 최저한세 자동 적용 | FR-16 | M5-03, F04/F-INC-06, 구간별 상세 | [PASS] |
| US-020 | 환급액 비교 보고서 8섹션 | FR-17 | M6-01~M6-05, OUT_REPORT_JSON | [PASS] |
| US-027 | RBAC 인증/권한 | FR-18 | SecurityConfig, 4단계 역할 | [PASS] |
| US-023 | 기준정보 관리 | FR-19 | REF_* 26개, 12.4절 갱신 프로세스 | [PASS] |
| **US-021** | **환급가산금 자동 계산** | **FR-20** | **M6-02, F31/F32/F-INC-09** | **[PASS]** |
| **US-022** | **경정청구서 초안 자동 생성** | **-** | **Phase 2로 이동** | **[PASS]** |

**주의**: 요구사항 문서에서 US-021(환급가산금)은 Phase 2로 분류되었으나, Plan v1.1에서 FR-20으로 Phase 1 MVP에 포함시켰다. 설계서에는 M6-02로 이미 설계되어 있어 설계 커버리지는 충족된다. 다만 요구사항의 US-022(경정청구서 초안)와 Plan의 US-022(지방소득세 안내)는 이름이 다르다 -- Plan에서 US-022의 내용이 변경(경정청구서 -> 지방소득세)된 것으로 보인다.

### 2.2 Phase 2 스토리 매핑 (해당 항목만)

| US ID | 스토리 | Phase 구분 | Design 반영 | 상태 |
|:-----:|--------|:---------:|:-----------:|:----:|
| US-014 | 사회보험료 SS30의4 | Phase 2 | M4-03 [Phase 2] 명시 | [PASS] |
| US-015 | 법인 전용 (이월결손금 등) | Phase 2 | M4-07~M4-15 디렉토리 존재 | [PASS] |
| US-016 | 개인 전용 (노란우산 등) | Phase 2 | P4-01~P4-12 디렉토리 존재 | [PASS] |
| US-024 | 상호배제 규칙 UI 관리 | Phase 2 | 미설계 (Phase 2) | [PASS] |
| US-025 | 대시보드 | Phase 2 | 미설계 (Phase 2) | [PASS] |
| US-026 | 사후관리 알림 | Phase 2 | 미설계 (Phase 2) | [PASS] |

**Phase 1 MVP 커버리지**: 21/21 FR, 19/19 Phase 1 Must Have US -- [PASS]

### 2.3 요구사항-Plan 스토리 ID 불일치 (v1.3 신규 발견)

| 항목 | 요구사항 문서 | Plan v1.1 | 상태 |
|------|-----------|----------|:----:|
| US-021 | 환급가산금 자동 계산 (EP-05, Phase 2) | US-021: 환급가산금 기본 계산 (Phase 1, 5SP) | [WARN] |
| US-022 | **경정청구서 초안 자동 생성** (EP-05, Phase 2) | US-022: **지방소득세 환급액 안내** (Phase 1, 3SP) | [WARN] |

**분석**: 요구사항 문서의 US-021/US-022와 Plan의 US-021/US-022가 다른 기능을 가리킨다. Plan v1.1에서 FR-20/FR-21 추가 시 Sprint 5 로드맵에 US-021(환급가산금)과 US-022(지방소득세)를 배정했으나, 요구사항 문서의 원래 US-022는 "경정청구서 초안 자동 생성"이다. 스토리 ID 재조정이 필요하다.

---

## 3. 스키마(schema.md) 대비 설계서 정합성

> Schema 문서: `docs/01-plan/schema.md` (63개 테이블, 5 레이어)
> Design 문서의 DB 관련 섹션

### 3.1 테이블 수/구조 정합성

| 항목 | schema.md | design.md | 상태 | 상세 |
|------|----------|----------|:----:|------|
| 총 테이블 수 | **63개** | **63개** | [PASS] | 12.2절에서 63개 명시 |
| Layer 1 (INP_) | 20개 | 20개 | [PASS] | 일치 |
| Layer 2 (SMR_) | 7개 | 7개 | [PASS] | 일치 |
| Layer 3 (OUT_) | 9개 | 9개 | [PASS] | 일치 |
| Layer 4 (REF_) | 26개 | 26개 | [PASS] | 일치 |
| Layer 5 (AUD_) | 1개 | 1개 | [PASS] | 일치 |

### 3.2 접두어 체계 정합성

| 항목 | schema.md 접두어 | design.md 접두어 | 상태 |
|------|----------------|-----------------|:----:|
| 입력 계층 | INP_ | INP_ | [PASS] |
| 요약 계층 | SMR_ | SMR_ | [PASS] |
| 출력 계층 | OUT_ | OUT_ | [PASS] |
| 기준정보 | REF_ | REF_ | [PASS] |
| 감사 로그 | AUD_ | AUD_ | [PASS] |

### 3.3 세목 분기 접미어 정합성

| 항목 | schema.md | design.md | 상태 |
|------|----------|----------|:----:|
| 법인 전용 | `_C_` (예: INP_C_CORP_BASIC) | INP_C_*, REF_C_* | [PASS] |
| 개인 전용 | `_I_` (예: INP_I_INC_BASIC) | INP_I_*, REF_I_* | [PASS] |
| 공통 | `_S_` (예: INP_S_EMPLOYEE_DETAIL) | INP_S_*, REF_S_* | [PASS] |

### 3.4 핵심 테이블 컬럼 정합성 검증

| 테이블 | schema.md 컬럼 | design.md 반영 | 상태 | 상세 |
|--------|--------------|:-------------:|:----:|------|
| INP_REQUEST | req_id, applicant_id, taxpayer_id, tax_type, tax_year, request_status, idempotency_key 등 15컬럼 | InpRequest.java 엔티티에 주요 컬럼 매핑 | [PASS] | |
| INP_RAW_DATASET | req_id, category, raw_json, schema_version, checksum 등 7컬럼 | InpRawDataset.java 엔티티에 매핑 | [PASS] | |
| SMR_BASIC_INFO | req_id, tax_type, corp_size 등 20컬럼 | SmrBasicInfo 엔티티 언급, 컬럼 상세는 schema.md 참조 위임 | [WARN] | 설계서에서 엔티티 전체 컬럼 정의 생략 |
| OUT_CREDIT_DETAIL | 17컬럼 상세 정의 | OutCreditDetail 엔티티 언급, schema.md 참조 위임 | [WARN] | 동일 |
| AUD_CALCULATION_LOG | 11컬럼 상세 정의 | AudCalculationLog 엔티티 언급, schema.md 참조 위임 | [PASS] | |

### 3.5 schema.md와 design.md의 구조적 불일치

| 이슈 | 상세 | 상태 |
|------|------|:----:|
| ~~SMR_VALIDATION_LOG~~ | ~~JPA 엔티티 디렉토리에 누락~~ | **RESOLVED** v1.2: SmrValidationLog.java 추가 |
| ~~SMR_PREP 테이블~~ | ~~entity/smr/ 하위에 미명시~~ | **RESOLVED** v1.2: SmrPrep.java 추가 |
| ~~SMR_ELIGIBILITY 테이블~~ | ~~엔티티 클래스 디렉토리 누락~~ | **RESOLVED** v1.2: SmrEligibility.java 추가 |

---

## 4. 설계서 내부 일관성

### 4.1 Java 버전 -- **RESOLVED (v1.1)**

| 위치 | v1.0 (수정 전) | v1.1 (수정 후) | 상태 |
|------|:-------------:|:-------------:|:----:|
| Plan 4.2 핵심 아키텍처 결정 | **Java 17** | **Java 17** | 기준 |
| Design 1.1 핵심 기술 스택 | ~~Java 1.8~~ | **Java 17 이상 (17+)** | [PASS] |
| Design 1.2 개발 환경 JDK | ~~Oracle JDK 1.8~~ | **Oracle JDK 17 / OpenJDK 17+** | [PASS] |
| Design 3. pom.xml `<java.version>` | ~~1.8~~ | **17** | [PASS] |

### 4.2 API 인증 방식 일관성 -- [PASS]

설계서 전체가 JWT 방식으로 통일. Plan과 일치. 참고1번의 API Key 방식에서 JWT로 개선한 것은 적절.

### 4.3 테이블 접두어 일관성 -- [PASS]

설계서 내부에서 INP_/SMR_/OUT_/REF_/AUD_ 접두어가 일관적으로 schema.md와 일치.

### 4.4 공식 수 불일치 (v1.3 신규 발견)

| 위치 | 기술 내용 | 상태 | 심각도 |
|------|---------|:----:|:------:|
| Design 2절 test 디렉토리 주석 (line 344) | `# 53개 공식 단위 테스트` | **[WARN]** | **Medium** |
| Design 1.1절 기술 스택 (line 56) | `56개 공식 단위 테스트 필수` | 기준 | - |
| Design 11.3절 전체 공식 목록 (line 1702) | `전체 계산 공식 목록 (56개)` | 기준 | - |
| Design 17.1절 테스트 계획 (line 2053) | `모든 계산 공식(56개)` | 기준 | - |

**분석**: 설계서 2절 디렉토리 구조의 test 디렉토리 주석에 "53개 공식 단위 테스트"로 기재되어 있다. 이는 v1.0에서 작성된 구 값으로, v1.2에서 56개로 통일할 때 이 주석을 갱신하지 않은 것이다. "53개"를 "56개"로 수정해야 한다.

### 4.5 공식 서브카운트 부정확 (v1.3 신규 발견)

| 위치 | 기술 내용 | 실제 | 상태 | 심각도 |
|------|---------|------|:----:|:------:|
| Design 11.3절 (line 1704) | `법인 F01~F41(**17개**)` | F01-F04(4) + F10-F19(10) + F30-F34(5) + F40-F41(2) = **21개** | **[WARN]** | Low |

**분석**: "F01~F41(17개)"에서 "17개"는 부정확하다. 실제 나열된 법인 공식은 21개이다. 다만 총합 56개라는 수치 자체는 참고1번의 "56개"와 일치하는 프로젝트 공인 수치로, 법인(F-series) 내부의 서브카운트 표기만 틀린 것이다. 총합에는 영향 없음.

### 4.6 API 엔드포인트 수 -- [PASS]

| 위치 | API 수 | 상태 |
|------|:------:|:----:|
| Plan 4.3 (v1.1) | 11개 (API-01~11) | [PASS] |
| Design 13.1 | 11개 (API-01~11) | [PASS] |
| 참고1번 4.2 | 11개 (API-01~11) | [PASS] |

### 4.7 TX-1/TX-2 범위 일관성 -- [PASS]

| 위치 | TX-1 범위 | TX-2 범위 | 상태 |
|------|---------|---------|:----:|
| Design 11.1 파이프라인 | M1-01~M1-12, P1-01~P1-08 | M3-PREP~M6-05 | [PASS] |
| Design 14.1 트랜잭션 경계 | 동일 | 동일 | [PASS] |

### 4.8 application/port 디렉토리 -- **RESOLVED (v1.2)**

인바운드 포트(CreateRequestUseCase, AnalyzeUseCase, ReportUseCase, ReferenceDataUseCase)와 아웃바운드 포트(PersistRequestPort, LoadSummaryPort, PersistCreditResultPort, LoadReferenceDataPort) 설계 완료.

### 4.9 Plan 내부 테이블 수 불일치 (v1.3 신규 발견)

| 위치 | 기술 내용 | 올바른 값 | 상태 | 심각도 |
|------|---------|----------|:----:|:------:|
| Plan 4.3 Clean Architecture 도표 (line 212) | `JPA Entity (83개 테이블)` | **63개** (schema.md 기준) | **[WARN]** | Medium |
| Plan 4.6 데이터베이스 설계 방향 (line 269) | `83개 테이블 + 뷰 1개` | **63개** (schema.md 기준) | **[WARN]** | Medium |
| Plan 9 다음 단계 (line 494) | `83개 테이블 DDL 확정` | **63개** (schema.md 기준) | **[WARN]** | Medium |

**분석**: Plan 문서의 세 곳에서 "83개 테이블"로 기재되어 있으나, schema.md(v1.0)에서 63개로 간소화되었고 설계서(v1.2)도 63개로 일치한다. Plan 문서가 참고1번의 83개 기준으로 작성된 후, schema.md 확정 시 Plan 내부의 테이블 수를 갱신하지 않은 것이다.

### 4.10 요구사항 문서 내부 불일치 (v1.3 신규 발견)

| 위치 | 기술 내용 | 올바른 값 | 상태 | 심각도 |
|------|---------|----------|:----:|:------:|
| requirements 5.2 (line 714) | `53개 공식 단위 테스트 100% 통과` | **56개** | **[WARN]** | Low |
| requirements 5.3 (line 725) | `RI_S_RAW_DATA INSERT ONLY` | INP_RAW_DATASET (현 schema.md) | **[WARN]** | Low |
| requirements 5.3 (line 726) | `AL_S_CALCULATION에 보관` | AUD_CALCULATION_LOG (현 schema.md) | **[WARN]** | Low |

**분석**: 요구사항 문서가 참고1번 기준으로 작성되어 구 접두어(RI_, AL_)와 구 공식 수(53개)를 사용하고 있다. schema.md 확정 후 요구사항 문서를 갱신하지 않은 것이다. 설계서 자체에는 영향 없으나, 문서 간 정합성을 위해 갱신이 필요하다.

---

## 5. 참고1번 대비 누락사항

> 참고1번: `refer-to-doc/참고1번-development-design-v2.md` (v2.1)
> Design 문서: `docs/02-design/features/Tax-refund.design.md`

### 5.1 참고1번에 있으나 설계서에 누락된 핵심 요소

| 항목 | 참고1번 위치 | 설계서 반영 | 상태 | 심각도 |
|------|-----------|:---------:|:----:|:-----:|
| M2 기준정보 모듈 | 참고1번 2.3 패키지: `m2/` | ReferenceDataService + port/out으로 대체 | [WARN] | Low |
| ~~RF_* 갱신 프로세스~~ | ~~참고1번 10.3~~ | v1.2: 12.4절 갱신 프로세스 추가 | **RESOLVED** | - |
| M4-10 결손금 소급공제 NPV | 참고1번 13.5 의사코드 | M4-08 + P4-09 존재, M4-10 파일명 미명시 | [WARN] | Low |
| 보고서 3종 상세 구조 | 참고1번 8.2 경영진용/세무대리인용/국세청용 | M6-04만 존재, 3종 구분 상세 없음 | [WARN] | Low |
| ~~F-COM-01 사후관리 추징액~~ | ~~참고1번 6.3~~ | v1.2: 11.3절에 추가 | **RESOLVED** | - |
| ~~F11 통합고용 추가공제~~ | ~~참고1번 6.1~~ | v1.2: 11.3절에 추가 | **RESOLVED** | - |
| ~~최저한세율 구간 상세~~ | ~~참고1번 6.5~~ | v1.2: 11.3절에 추가 | **RESOLVED** | - |

### 5.2 참고1번에서 의도적으로 변경된 항목 (적절한 변경)

| 항목 | 참고1번 | 설계서 | 판단 |
|------|-------|-------|:----:|
| 테이블 수 | 83개 + 뷰 1 | 63개 | [PASS] -- schema.md 기반 간소화 |
| 접두어 | RI_/SV_/CO_/RF_/AL_ | INP_/SMR_/OUT_/REF_/AUD_ | [PASS] -- 직관성 향상 |
| API 인증 | API Key (X-API-Key) | JWT (Authorization: Bearer) | [PASS] -- Plan 방향 일치 |
| 기준정보 접두어 | RF_ | REF_ | [PASS] -- schema.md 일치 |

### 5.3 참고1번의 계산 공식 커버리지 -- **전체 포함 (v1.2)**

| 공식 그룹 | 참고1번 | 설계서 v1.2 반영 | 상태 |
|----------|-------|:---------:|:----:|
| F01~F04 (과세표준/산출세액/최저한세) | O | O | [PASS] |
| F10~F13 (통합고용/투자) | O | O (F11 추가 포함) | [PASS] |
| F14~F19 (창업/중소특별/R&D/사보) | O | O | [PASS] |
| F30~F34 (환급액/가산금) | O | O | [PASS] |
| F40~F41 (농특세) | O | O | [PASS] |
| F-INC-01~F-INC-11 | O | O (전체 11개) | [PASS] |
| F-COM-01 (사후관리 추징) | O | O | [PASS] |

---

## 6. 용어/명명 일관성

### 6.1 테이블명 일관성

| 구분 | schema.md | design.md | 참고1번 | 상태 |
|------|----------|----------|--------|:----:|
| 요청 마스터 | INP_REQUEST | INP_REQUEST (InpRequest.java) | RI_S_REQUEST | [PASS] |
| JSON 원본 | INP_RAW_DATASET | INP_RAW_DATASET (InpRawDataset.java) | RI_S_RAW_DATA | [PASS] |
| 기초정보 요약 | SMR_BASIC_INFO | SMR_BASIC_INFO (SmrBasicInfo.java) | SV_S_BASIC | [PASS] |
| 고용 요약 | SMR_EMPLOYEE | SMR_EMPLOYEE (SmrEmployee.java) | SV_S_EMPLOYEE | [PASS] |
| 산출 결과 | OUT_CREDIT_DETAIL | OUT_CREDIT_DETAIL (OutCreditDetail.java) | CO_S_CREDIT_DETAIL | [PASS] |
| 보고서 JSON | OUT_REPORT_JSON | OUT_REPORT_JSON (OutReportJson.java) | CO_S_REPORT_JSON | [PASS] |

### 6.2 엔티티-테이블 명명 규칙

규칙: `{테이블접두어}_{테이블명}` -> `{PascalCase접두어}{PascalCase테이블명}` -- 전체 일관 [PASS]

| 테이블 | 엔티티 클래스명 | 상태 |
|--------|-------------|:----:|
| inp_request | InpRequest | [PASS] |
| inp_raw_dataset | InpRawDataset | [PASS] |
| smr_basic_info | SmrBasicInfo | [PASS] |
| smr_employee | SmrEmployee | [PASS] |
| smr_deduction_item | SmrDeductionItem | [PASS] |
| smr_financial | SmrFinancial | [PASS] |
| smr_eligibility | SmrEligibility | [PASS] |
| smr_prep | SmrPrep | [PASS] |
| smr_validation_log | SmrValidationLog | [PASS] |
| out_credit_detail | OutCreditDetail | [PASS] |
| out_combination | OutCombination | [PASS] |
| out_refund | OutRefund | [PASS] |
| out_report_json | OutReportJson | [PASS] |
| aud_calculation_log | AudCalculationLog | [PASS] |
| ref_tax_rate_bracket | RefTaxRateBracket | [PASS] |

### 6.3 조항 코드 표기

| 조항 | schema.md | design.md | 참고1번 | 상태 |
|------|----------|----------|--------|:----:|
| 통합투자 | SS24 | SS24 | SS24 | [PASS] |
| 통합고용 | SS29_8 | SS29_8 | SS29의8 | [PASS] (코드 내 일관) |
| 사회보험 | SS30_4 | SS30_4 | SS30의4 | [PASS] (코드 내 일관) |
| 창업감면 | SS6 | SS6 | SS6 | [PASS] |
| 중소특별 | SS7 | SS7 | SS7 | [PASS] |
| R&D | SS10 | SS10 | SS10 | [PASS] |

### 6.4 Enum 명명

| Enum | design.md | schema.md | 상태 |
|------|----------|----------|:----:|
| TaxType | CORP / INC | CORP / INC | [PASS] |
| CorpSize | SMALL / MEDIUM / MID_LARGE / LARGE | 동일 | [PASS] |
| CapitalZone | OVER_CONCENTRATION / GROWTH_MGMT / NON_CAPITAL / DEPOPULATION | 동일 | [PASS] |
| RequestStatus | received ~ completed / error (10단계) | 동일 | [PASS] |
| CreditType | EXEMPTION / CREDIT / DEDUCTION | 동일 | [PASS] |

### 6.5 용어 사용 규칙 준수

| 규칙 (schema.md 1.4) | 설계서 준수 | 상태 |
|---------------------|:----------:|:----:|
| 코드에서 영문: camelCase | O (reqId, taxType, calculatedTax) | [PASS] |
| DB 컬럼: snake_case | O (req_id, tax_type, calculated_tax) | [PASS] |
| API 응답: camelCase | O (reqId, datasetsReceived, createdAt) | [PASS] |
| 세목 코드: CORP/INC | O (전체 문서 일관) | [PASS] |

---

## 7. 종합 이슈 요약

### 7.1 Critical (구현 불가능) -- 0건

| No | 이슈 | 상태 |
|----|------|:----:|
| ~~C-01~~ | ~~Java 버전 불일치~~ | **RESOLVED** v1.1 |

### 7.2 Warning (개선 필요) -- 기존 3건(Open) + 신규 6건 = **9건**

| No | 이슈 | 심각도 | 위치 | 상태 |
|----|------|:------:|------|:----:|
| ~~W-01~~ | ~~SMR 엔티티 클래스 누락~~ | ~~High~~ | ~~design.md 2.~~ | **RESOLVED** v1.2 |
| ~~W-02~~ | ~~RF_* 갱신 프로세스 미설계~~ | ~~High~~ | ~~design.md~~ | **RESOLVED** v1.2 |
| ~~W-03~~ | ~~계산 공식 전체 목록 미포함~~ | ~~High~~ | ~~design.md 11.3~~ | **RESOLVED** v1.2 |
| ~~W-04~~ | ~~농특세 공식 미포함~~ | ~~Medium~~ | ~~design.md 11.3~~ | **RESOLVED** v1.2 |
| ~~W-05~~ | ~~F-COM-01 미포함~~ | ~~Medium~~ | ~~design.md 11.3~~ | **RESOLVED** v1.2 |
| ~~W-06~~ | ~~M4-03 Phase 2 미표시~~ | ~~Medium~~ | ~~design.md 2.~~ | **RESOLVED** v1.2 |
| ~~W-07~~ | ~~port 패키지 누락~~ | ~~Medium~~ | ~~design.md 2.~~ | **RESOLVED** v1.2 |
| ~~W-08~~ | ~~부하 테스트 미설계~~ | ~~Medium~~ | ~~design.md~~ | **RESOLVED** v1.2 |
| ~~W-09~~ | ~~E2E 시나리오 미정의~~ | ~~Medium~~ | ~~design.md~~ | **RESOLVED** v1.2 |
| W-10 | SonarQube CI/CD 파이프라인 상세 설정 미포함 | Low | design.md 17.4절 | Open |
| ~~W-11~~ | ~~SLA 99.5% 미설계~~ | ~~Medium~~ | ~~design.md~~ | **RESOLVED** v1.2 |
| W-12 | M2 기준정보 모듈 패키지(m2/) 미포함 (ReferenceDataService로 대체) | Low | design.md 2절 | Open |
| W-13 | 보고서 3종(경영진/세무대리인/국세청) 상세 구조 미포함 | Low | design.md 전체 | Open |
| ~~W-14~~ | ~~최저한세율+R&D 배제 상세 미포함~~ | ~~Medium~~ | ~~design.md 11.3~~ | **RESOLVED** v1.2 |
| ~~W-15~~ | ~~Plan API 수 불일치~~ | ~~Low~~ | ~~Plan 4.3~~ | **RESOLVED** Plan v1.1 |
| **W-16** | **Design 2절 test 주석 "53개 공식" -> "56개"로 미갱신** | **Medium** | design.md line 344 | **New** |
| **W-17** | **Design 11.3절 "F01~F41(17개)" -> 실제 21개 서브카운트 표기 오류** | **Low** | design.md line 1704 | **New** |
| **W-18** | **Plan 6.1 DoD "19개 기능" -> FR-20/FR-21 추가로 "21개"여야 함** | **Medium** | Plan line 388 | **New** |
| **W-19** | **Plan 4.3/4.6/9절 "83개 테이블" -> schema.md 확정 후 "63개"로 미갱신** | **Medium** | Plan line 212, 269, 494 | **New** |
| **W-20** | **요구사항 US-021/US-022 vs Plan US-021/US-022 스토리 ID 불일치** | **Medium** | Plan Sprint 5, requirements 3.5 | **New** |
| **W-21** | **요구사항 5.2절 "53개 공식" -> "56개"로 미갱신, 구 접두어(RI_/AL_) 잔존** | **Low** | requirements line 714, 725, 726 | **New** |

### 7.3 Info (참고) -- 3건

| No | 이슈 | 위치 |
|----|------|------|
| I-01 | 참고1번 접두어(RI_/SV_/CO_/RF_/AL_) -> INP_/SMR_/OUT_/REF_/AUD_로 의도적 변경. schema.md 11.2에 변경 이유 명시 | schema.md 11.2 |
| I-02 | 테이블 수 83개(참고1번) -> 63개(schema.md/설계서)로 간소화. schema.md 11.2에 변경 이유 명시 | schema.md 11.2 |
| I-03 | API 인증이 API Key(참고1번) -> JWT(설계서)로 변경. Plan 방향과 일치하는 적절한 변경 | design.md 15.1 |

---

## 8. 권고사항

### 8.1 즉시 수정 필요 (설계서 내부 -- 10분 이내)

| 순위 | 권고 | 대상 파일 | 예상 공수 |
|:----:|------|---------|:--------:|
| 1 | **Design 2절 test 주석 "53개" -> "56개" 수정** (W-16) | design.md line 344 | 1분 |
| 2 | **Design 11.3절 "F01~F41(17개)" -> "F01~F41(21개)" 수정** (W-17) | design.md line 1704 | 1분 |

### 8.2 우선 보완 권고 (Plan/요구사항 문서 -- 30분 이내)

| 순위 | 권고 | 대상 파일 | 예상 공수 |
|:----:|------|---------|:--------:|
| 3 | **Plan 6.1 DoD "19개" -> "21개" 수정** (W-18) | Plan line 388 | 1분 |
| 4 | **Plan 4.3/4.6/9절 "83개 테이블" -> "63개" 수정** (W-19) | Plan line 212, 269, 494 | 5분 |
| 5 | **Plan Sprint 5 US-021/US-022 스토리 ID를 요구사항과 정합시키거나, 별도 스토리 ID(US-028/US-029) 부여** (W-20) | Plan, requirements | 15분 |
| 6 | **요구사항 5.2절 "53개" -> "56개", 구 접두어 -> 현 접두어 갱신** (W-21) | requirements | 10분 |

### 8.3 향후 보완 권고 (구현 진행 중 보완 가능)

| 순위 | 권고 | 대상 파일 |
|:----:|------|---------|
| 7 | SonarQube Quality Gate 상세 설정 추가 (W-10) | design.md 17.4절 |
| 8 | 보고서 3종(경영진/세무대리인/국세청) 상세 구조 추가 (W-13) | design.md |
| 9 | M2 기준정보 모듈 설계 여부 결정 (W-12) | design.md 2절 |

---

## 체크리스트 결과 요약

| 검증 항목 | 결과 | 비고 |
|----------|:----:|------|
| Plan 대비 기능 요구사항 커버리지 (FR-01~**FR-21**) | **PASS (21/21)** | v1.3: FR-20/FR-21 매핑 추가 |
| Plan 대비 비기능 요구사항 커버리지 | PASS | SLA 99.5%, JMeter 부하테스트 설계 |
| **요구사항(US-001~027) 대비 Phase 1 커버리지** | **PASS** | **v1.3 신규: 19 Must Have US + FR-20/FR-21 커버** |
| Schema 테이블 수/구조 정합성 | PASS | 63개, 5 레이어 일치 |
| Schema 접두어/세목 분기 정합성 | PASS | INP_/SMR_/OUT_/REF_/AUD_, _C_/_I_/_S_ 일치 |
| Schema 컬럼 레벨 정합성 | WARN | 일부 엔티티 전체 컬럼 미기재 (schema 참조 위임) |
| Schema 엔티티 클래스 누락 | PASS | v1.2 해소 |
| 내부 Java 버전 일관성 | PASS | v1.1에서 통일 |
| 내부 API 인증 방식 일관성 | PASS | JWT 통일 |
| 내부 접두어 일관성 | PASS | INP_/SMR_/OUT_/REF_/AUD_ 통일 |
| 내부 TX 범위 일관성 | PASS | TX-1 (M1~P1), TX-2 (M3~M6) 일관 |
| **내부 공식 수 일관성** | **WARN** | **v1.3: test 주석 "53개" 잔존 (W-16)** |
| 참고1번 대비 RF_* 갱신 프로세스 | PASS | v1.2 해소 |
| 참고1번 대비 계산 공식 완전성 | PASS | 56개 전체 포함 |
| 용어 일관성 | PASS | CORP/INC, camelCase/snake_case 준수 |
| 엔티티/테이블 명명 일관성 | PASS | PascalCase 변환 규칙 일관 |
| 조항 코드 일관성 | PASS | SS24, SS29_8 등 코드 내 일관 |
| **Plan 내부 수치 일관성** | **WARN** | **v1.3: "83개"/"19개" 미갱신 (W-18, W-19)** |
| **요구사항-Plan 스토리 ID 정합성** | **WARN** | **v1.3: US-021/022 불일치 (W-20)** |

---

## 점수 산출 근거

| 카테고리 | 배점 | v1.2 득점 | v1.3 득점 | v1.3 변동 사유 |
|---------|:----:|:--------:|:--------:|---------|
| Plan 대비 커버리지 | 20 | 20 | **19** | **20** | **20** | v1.4: DoD 21개 수정(+1) |
| 요구사항 대비 커버리지 | 10 | (미평가) | **8** | **8** | **10** | v1.5: US ID 통일, US-028 신설(+2) |
| Schema 정합성 | 15 | 15 | **15** | **15** | **15** | 변동 없음 |
| 내부 일관성 | 25 | 24 | **22** | **25** | **25** | 변동 없음 |
| 참고1번 대비 완전성 | 15 | 15 | **15** | **15** | **15** | 변동 없음 |
| 용어/명명 일관성 | 15 | 11 | **12** | **12** | **12** | 요구사항 접두어 통일(+1), 잔여 Low(-1) |
| **합계** | **100** | **93** | **91** | **95** | **97** | v1.5: W-20~W-21 수정으로 +2 |

---

## 최종 판정

**완전성 점수: 97/100** (v1.0: 78 -> v1.1: 84 -> v1.2: 93 -> v1.3: 91 -> v1.4: 95 -> v1.5: 97)

> 점수 90~100 구간: **구현 착수 가능 (PDCA 기준 통과)**

### v1.3 전수 검증 결과 요약

**긍정적 발견 (기존 v1.2 보완 사항 유지)**:
- Warning 12건 해소 상태 유지 (v1.1에서 1건, v1.2에서 11건)
- FR-20(환급가산금) / FR-21(지방소득세) -- 설계서에 M6-02, M6-03, F31~F33, F-INC-09으로 이미 반영 확인
- US-001~027 전수 매핑 완료 -- Phase 1 Must Have 19개 + FR-20/FR-21 모두 설계 커버리지 충족
- 56개 공식 전체 목록 (11.3절), 10규칙 상호배제 (11.1절), RF_* 갱신 프로세스 (12.4절), E2E 10건 (17.2절), SLA 99.5% (18절) 모두 양호

**신규 발견 Warning 6건**:
1. ~~**W-16** (Medium): Design 2절 test 디렉토리 주석 "53개 공식" -- "56개"로 수정 필요~~ **RESOLVED** ✅ v1.4: 56개로 수정 완료
2. ~~**W-17** (Low): Design 11.3절 서브카운트 "17개" -- 실제 21개~~ **RESOLVED** ✅ v1.4: 21개로 수정 + 비연속 번호 설명 추가
3. ~~**W-18** (Medium): Plan 6.1 DoD "19개 기능" -- "21개"로 수정 필요~~ **RESOLVED** ✅ v1.4: 21개로 수정 완료
4. ~~**W-19** (Medium): Plan 3곳의 "83개 테이블" -- "63개"로 수정 필요~~ **RESOLVED** ✅ v1.4: 63개로 4곳 수정 + 접두어 INP_/SMR_/OUT_/AUD_/REF_로 통일
5. ~~**W-20** (Medium): 요구사항 US-021/022와 Plan US-021/022 스토리 ID 불일치~~ **RESOLVED** ✅ v1.5: US-021(환급가산금) Phase 1 승격 반영, US-028(지방소득세 안내) 신설, US-022(경정청구서 초안) Phase 2 유지, 의존성 다이어그램 갱신
6. ~~**W-21** (Low): 요구사항 문서의 구 수치("53개")/구 접두어(RI_/AL_) 잔존~~ **RESOLVED** ✅ v1.5: 53→56개 수정, RI_S_RAW_DATA→INP_RAW_DATASET, AL_S_CALCULATION→AUD_CALCULATION_LOG, SV_*→SMR_*, RF_S_*→REF_S_* 접두어 통일

**잔여 기존 Warning 3건** (Low, 변동 없음):
- W-10: SonarQube 상세 설정
- W-12: M2 모듈 패키지
- W-13: 보고서 3종 상세

**총 Open Warning: 3건** (Low 3건) -- Critical 0건, Medium 0건

v1.4에서 W-16~W-19 4건, v1.5에서 W-20~W-21 2건이 수정 완료되었다. 잔여 3건은 모두 Low로 구현 착수에 차단 없음.

---

## 버전 이력

| 버전 | 일자 | 변경 내용 | 작성자 |
|-----|------|---------|--------|
| 1.0 | 2026-02-18 | 최초 검증 -- 5개 검증 항목 전수 수행. 점수 78 | Design Validation Agent |
| 1.1 | 2026-02-18 | Critical 수정 반영 -- C-01 Java 17 수정(RESOLVED), 상호배제 10규칙 테이블 추가 반영. 점수 78->84 | Design Validation Agent |
| 1.2 | 2026-02-18 | Warning 9건 보완 반영 -- SMR 엔티티 3개, RF_* 갱신 프로세스, 공식 56개 전체, 농특세/최저한세 상세, F-COM-01, port 패키지, 테스트 계획, SLA 설계, M4-03 Phase 2 명시. 점수 84->93 | Design Validation Agent |
| 1.3 | 2026-02-18 | 전수 검증 -- (1) FR-20/FR-21 매핑 추가 (21/21 PASS) (2) US-001~027 전수 매핑 추가 (3) Plan 내부 수치 불일치 3건 발견 (83개/19개/83개) (4) Design test 주석 53개 잔존 발견 (5) 요구사항-Plan US ID 불일치 발견 (6) 요구사항 구접두어/구수치 잔존 발견. 점수 93->91 (배점 재조정 + 신규 Warning 6건) | Design Validation Agent |
| 1.4 | 2026-02-18 | W-16~W-19 수정 반영 -- (1) Design test 주석 53→56개 (W-16) (2) Design F01~F41 카운트 17→21개 + 비연속 설명 (W-17) (3) Plan DoD 19→21개 (W-18) (4) Plan 83→63개 테이블 4곳 + 접두어 INP_/SMR_/OUT_/AUD_/REF_로 통일 (W-19). 점수 91->95 | Design Validation Agent |
| 1.5 | 2026-02-18 | W-20~W-21 수정 반영 -- (1) 요구사항서 US-021 Phase 1 승격 반영, US-028(지방소득세 안내) 신설, Phase 1 Must Have 표·의존성 다이어그램 갱신 (W-20) (2) 요구사항서 53→56개, 접두어 6곳 통일: RI_→INP_, AL_→AUD_, SV_→SMR_, RF_→REF_ (W-21) (3) 계획서 US-022→US-028 재번호, 의존성 다이어그램 갱신. 점수 95->97 | Design Validation Agent |
