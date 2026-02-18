> **[ARCHIVE]** 2026-02-18 아카이브. 문서 내 경로(`docs/01-plan/`, `docs/02-design/`, `review-docs/`)는 아카이브 전 구조 기준. 실제 위치: `docs/archive/2026-02/Tax-refund/`

# Tax-refund 설계서 — 통합 경정청구 환급액 산출 시스템

> **Summary**: 법인(법인세)과 개인사업자(종합소득세) 경정청구 환급액 극대화 통합 점검 시스템 상세 설계
>
> **Project**: TaxServiceENTEC
> **Version**: v1.4
> **Author**: Design (PDCA)
> **Date**: 2026-02-18
> **Status**: Draft
> **참조 문서**:
>   - 계획서: `docs/01-plan/features/Tax-refund.plan.md`
>   - 스키마: `docs/01-plan/schema.md` (63개 테이블, 5계층)
>   - 참고1번: `refer-to-doc/참고1번-development-design-v2.md` (방향성 참조)

---

## 목차

1. [기술 스택 및 개발 환경](#1-기술-스택-및-개발-환경)
2. [프로젝트 디렉토리 구조](#2-프로젝트-디렉토리-구조)
3. [pom.xml 의존성 설계](#3-pomxml-의존성-설계)
4. [Spring Boot Application 및 설정 파일](#4-spring-boot-application-및-설정-파일)
5. [Config 파일 설계](#5-config-파일-설계)
6. [공통 엔티티/DTO 설계](#6-공통-엔티티dto-설계)
7. [예외 처리 체계](#7-예외-처리-체계)
8. [공통 요소 및 유틸리티 (Common & Utils)](#8-공통-요소-및-유틸리티)
9. [접근 권한 / 로깅 / 감사 기능](#9-접근-권한--로깅--감사-기능)
10. [UI 메시지 / 오류 메시지 / 사용자 메시지](#10-ui-메시지--오류-메시지--사용자-메시지)
11. [환급액 계산 로직 상세 설계](#11-환급액-계산-로직-상세-설계)
12. [데이터베이스 테이블 설계](#12-데이터베이스-테이블-설계)
13. [REST API 설계](#13-rest-api-설계)
14. [트랜잭션 및 동시성 설계](#14-트랜잭션-및-동시성-설계)
15. [보안 설계](#15-보안-설계)
16. [소스코드 주석 컨벤션](#16-소스코드-주석-컨벤션)
17. [테스트 계획](#17-테스트-계획)
18. [가용성 및 모니터링 설계](#18-가용성-및-모니터링-설계)
19. [구현 순서](#19-구현-순서)
20. [용어 참조](#20-용어-참조)

---

## 1. 기술 스택 및 개발 환경

### 1.1 핵심 기술 스택

| 항목 | 선정 기술 | 버전 | 근거 |
|------|----------|------|------|
| **Framework** | Spring Boot | 3.x (3.2+) | 엔터프라이즈급 DI, 트랜잭션, Auto Configuration |
| **Language** | Java | 17 이상 (17+) | Spring Boot 3.x 최소 요구사항, Records/Sealed 지원, BigDecimal 정밀 계산 |
| **ORM** | JPA (Hibernate) | 6.x | 엔티티 관리, CRUD 자동화 |
| **SQL Mapper** | MyBatis | 3.5+ | 복잡 집계 쿼리, 동적 SQL, 보고서 조회 |
| **Database** | PostgreSQL | 15+ | JSONB 컬럼, 부분 유니크 인덱스, 풍부한 함수 |
| **Build** | Maven | 3.9+ | pom.xml 의존성 관리 |
| **API** | REST (JSON) | OpenAPI 3.0 | 단순성, 국세청 표준 준수 |
| **인증** | Spring Security + JWT | — | req_id 토큰화, RBAC 4단계 |
| **테스트** | JUnit 5 + AssertJ | — | 57개 공식 단위 테스트 필수 |
| **로깅** | Logback + SLF4J | — | 구조화 로그, req_id 추적 |
| **문서** | Swagger (springdoc) | — | OpenAPI 3.0 자동 생성 |

### 1.2 개발 환경

| 항목 | 설정 |
|------|------|
| JDK | Oracle JDK 17 또는 OpenJDK 17+ |
| IDE | IntelliJ IDEA / Eclipse (Spring Tool Suite) |
| DB 클라이언트 | DBeaver / pgAdmin |
| 버전 관리 | Git (feature 브랜치 전략) |
| 코드 스타일 | Google Java Style + Checkstyle |
| 정적 분석 | SonarQube |

---

## 2. 프로젝트 디렉토리 구조

```
tax-service-entec/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/com/taxservice/entec/
│   │   │   │
│   │   │   ├── TaxServiceEntecApplication.java          # Spring Boot Main
│   │   │   │
│   │   │   ├── presentation/                            # [계층 1] API 표현 계층
│   │   │   │   ├── controller/
│   │   │   │   │   ├── RequestController.java           # API-01: 요청 접수
│   │   │   │   │   ├── AnalyzeController.java           # API-02: 점검 실행
│   │   │   │   │   ├── StatusController.java            # API-03: 상태 조회
│   │   │   │   │   ├── ReportController.java            # API-04~06: 보고서
│   │   │   │   │   ├── ReferenceController.java         # API-07: 기준정보
│   │   │   │   │   └── AuditController.java             # API-11: 감사 로그
│   │   │   │   ├── dto/
│   │   │   │   │   ├── request/
│   │   │   │   │   │   ├── AmendmentClaimRequest.java   # 경정청구 요청 DTO
│   │   │   │   │   │   ├── DatasetItem.java             # 개별 데이터셋 DTO
│   │   │   │   │   │   └── AnalyzeRequest.java          # 점검 실행 요청 DTO
│   │   │   │   │   └── response/
│   │   │   │   │       ├── RequestResponse.java         # 요청 접수 응답
│   │   │   │   │       ├── StatusResponse.java          # 상태 조회 응답
│   │   │   │   │       ├── ReportResponse.java          # 보고서 응답
│   │   │   │   │       ├── CreditDetailResponse.java    # 공제/감면 상세 응답
│   │   │   │   │       └── ErrorResponse.java           # 표준 오류 응답
│   │   │   │   └── advice/
│   │   │   │       └── GlobalExceptionHandler.java      # 통합 예외 처리
│   │   │   │
│   │   │   ├── application/                             # [계층 2] 유스케이스 계층
│   │   │   │   ├── service/
│   │   │   │   │   ├── AmendmentClaimService.java       # 오케스트레이터 (메인)
│   │   │   │   │   ├── RequestService.java              # 요청 접수 서비스
│   │   │   │   │   ├── AnalyzeService.java              # 분석 오케스트레이션
│   │   │   │   │   ├── ReportService.java               # 보고서 생성 서비스
│   │   │   │   │   └── ReferenceDataService.java        # 기준정보 서비스
│   │   │   │   ├── port/                               # 포트 인터페이스
│   │   │   │   │   ├── in/                             # 인바운드 포트 (유스케이스)
│   │   │   │   │   │   ├── CreateRequestUseCase.java   # 요청 생성
│   │   │   │   │   │   ├── AnalyzeUseCase.java         # 분석 실행
│   │   │   │   │   │   ├── ReportUseCase.java          # 보고서 조회
│   │   │   │   │   │   └── ReferenceDataUseCase.java   # 기준정보 관리
│   │   │   │   │   └── out/                            # 아웃바운드 포트 (SPI)
│   │   │   │   │       ├── PersistRequestPort.java     # 요청 저장
│   │   │   │   │       ├── LoadSummaryPort.java        # 요약 조회
│   │   │   │   │       ├── PersistCreditResultPort.java # 산출 결과 저장
│   │   │   │   │       └── LoadReferenceDataPort.java  # 기준정보 조회
│   │   │   │   └── mapper/
│   │   │   │       └── DtoMapper.java                   # DTO ↔ Domain 매핑
│   │   │   │
│   │   │   ├── domain/                                  # [계층 3] 도메인 계층
│   │   │   │   ├── model/
│   │   │   │   │   ├── enums/
│   │   │   │   │   │   ├── TaxType.java                 # CORP / INC
│   │   │   │   │   │   ├── CorpSize.java                # SMALL/MEDIUM/MID_LARGE/LARGE
│   │   │   │   │   │   ├── CapitalZone.java             # 수도권 4구분
│   │   │   │   │   │   ├── RequestStatus.java           # 요청 상태 (10단계)
│   │   │   │   │   │   ├── CreditType.java              # EXEMPTION/CREDIT/DEDUCTION
│   │   │   │   │   │   ├── ValidationSeverity.java      # ERROR/WARNING
│   │   │   │   │   │   └── RiskLevel.java               # HIGH/MEDIUM/LOW
│   │   │   │   │   ├── vo/
│   │   │   │   │   │   ├── ReqId.java                   # 요청 ID Value Object
│   │   │   │   │   │   ├── MoneyAmount.java             # 금액 (절사 정책 내장)
│   │   │   │   │   │   ├── TaxRate.java                 # 세율 (소수점 3자리 절사)
│   │   │   │   │   │   └── ProvisionCode.java           # 조항 코드 (SS24 등)
│   │   │   │   │   └── entity/
│   │   │   │   │       ├── InpRequest.java              # INP_REQUEST 엔티티
│   │   │   │   │       ├── InpRawDataset.java           # INP_RAW_DATASET 엔티티
│   │   │   │   │       ├── SmrBasicInfo.java            # SMR_BASIC_INFO 엔티티
│   │   │   │   │       ├── SmrEmployee.java             # SMR_EMPLOYEE 엔티티
│   │   │   │   │       ├── SmrDeductionItem.java        # SMR_DEDUCTION_ITEM 엔티티
│   │   │   │   │       ├── SmrFinancial.java            # SMR_FINANCIAL 엔티티
│   │   │   │   │       ├── SmrEligibility.java          # SMR_ELIGIBILITY 엔티티 (적격성 판정 결과)
│   │   │   │   │       ├── SmrPrep.java                 # SMR_PREP 엔티티 (데이터 준비 결과)
│   │   │   │   │       ├── SmrValidationLog.java        # SMR_VALIDATION_LOG 엔티티 (107규칙 검증 결과)
│   │   │   │   │       ├── OutCreditDetail.java         # OUT_CREDIT_DETAIL 엔티티
│   │   │   │   │       ├── OutCombination.java          # OUT_COMBINATION 엔티티
│   │   │   │   │       ├── OutRefund.java               # OUT_REFUND 엔티티
│   │   │   │   │       ├── OutReportJson.java           # OUT_REPORT_JSON 엔티티
│   │   │   │   │       ├── AudCalculationLog.java       # AUD_CALCULATION_LOG 엔티티
│   │   │   │   │       └── ref/                         # REF_* 기준정보 엔티티
│   │   │   │   │           ├── RefTaxRateBracket.java
│   │   │   │   │           ├── RefMinTaxRate.java
│   │   │   │   │           ├── RefMutualExclusion.java
│   │   │   │   │           └── ... (26개 REF 테이블)
│   │   │   │   ├── repository/
│   │   │   │   │   ├── RequestRepository.java           # 요청 Repository 인터페이스
│   │   │   │   │   ├── RawDatasetRepository.java
│   │   │   │   │   ├── SummaryRepository.java
│   │   │   │   │   ├── CreditResultRepository.java
│   │   │   │   │   ├── CombinationRepository.java
│   │   │   │   │   ├── ReferenceDataRepository.java
│   │   │   │   │   └── AuditLogRepository.java
│   │   │   │   └── service/
│   │   │   │       ├── module/                          # M 모듈 (공통)
│   │   │   │       │   ├── m1/
│   │   │   │       │   │   ├── RequestCreationService.java      # M1-01
│   │   │   │       │   │   ├── RawDataStorageService.java       # M1-02
│   │   │   │       │   │   ├── SummaryGenerationService.java    # M1-03
│   │   │   │       │   │   ├── InvestmentValidator.java         # M1-04
│   │   │   │       │   │   ├── StartupValidator.java            # M1-05
│   │   │   │       │   │   ├── RdExpenseValidator.java          # M1-06
│   │   │   │       │   │   ├── LossCarryforwardValidator.java   # M1-07
│   │   │   │       │   │   ├── CreditCarryforwardValidator.java # M1-08
│   │   │   │       │   │   ├── ExistingDeductionValidator.java  # M1-09
│   │   │   │       │   │   ├── BranchLocationValidator.java     # M1-10
│   │   │   │       │   │   ├── InterimTaxValidator.java         # M1-11
│   │   │   │       │   │   └── DividendIncomeValidator.java     # M1-12
│   │   │   │       │   ├── m3/
│   │   │   │       │   │   ├── DataPrepService.java             # M3-PREP
│   │   │   │       │   │   ├── HardFailCheckService.java        # M3-00
│   │   │   │       │   │   ├── DeadlineCheckService.java        # M3-01
│   │   │   │       │   │   ├── SmeEligibilityService.java       # M3-02
│   │   │   │       │   │   ├── LocationCheckService.java        # M3-03
│   │   │   │       │   │   ├── EmployeeCountService.java        # M3-04
│   │   │   │       │   │   ├── IndustryEligibilityService.java  # M3-05
│   │   │   │       │   │   └── SettlementCheckService.java      # M3-06
│   │   │   │       │   ├── m4/                                  # 25개 CreditCalculator
│   │   │   │       │   │   ├── CreditCalculator.java            # Strategy 인터페이스
│   │   │   │       │   │   ├── CreditCalculatorRegistry.java    # Registry
│   │   │   │       │   │   ├── CalculationContext.java          # 계산 컨텍스트
│   │   │   │       │   │   ├── InvestmentCreditCalc.java        # M4-01 §24
│   │   │   │       │   │   ├── EmploymentCreditCalc.java        # M4-02 §29의8
│   │   │   │       │   │   ├── SocialInsuranceCreditCalc.java   # M4-03 §30의4 [Phase 2 — 2024년 일몰]
│   │   │   │       │   │   ├── StartupExemptionCalc.java        # M4-04 §6
│   │   │   │       │   │   ├── SmeSpecialExemptionCalc.java     # M4-05 §7
│   │   │   │       │   │   ├── RdCreditCalc.java                # M4-06 §10
│   │   │   │       │   │   ├── LossCarryforwardReviewCalc.java  # M4-08 §13
│   │   │   │       │   │   ├── ForeignTaxCreditCalc.java       # M4-09 §57 (법인 외국납부)
│   │   │   │       │   │   ├── CorpRestructuringCalc.java      # M4-10 §44~47 (기업구조조정 과세이연)
│   │   │   │       │   │   └── ConsolidatedTaxCalc.java        # M4-11 §76의8 (연결납세)
│   │   │   │       │   ├── m5/
│   │   │   │       │   │   ├── MutualExclusionService.java      # M5-01
│   │   │   │       │   │   ├── CombinationSearchService.java    # M5-02 B&B/Greedy
│   │   │   │       │   │   ├── MinimumTaxService.java           # M5-03
│   │   │   │       │   │   ├── AgriculturalTaxService.java      # M5-04
│   │   │   │       │   │   └── ApplicationOrderService.java     # M5-05
│   │   │   │       │   ├── m6/
│   │   │   │       │   │   ├── RefundComparisonService.java     # M6-01
│   │   │   │       │   │   ├── RefundInterestService.java       # M6-02
│   │   │   │       │   │   ├── LocalTaxGuideService.java        # M6-03
│   │   │   │       │   │   ├── ReportGenerationService.java     # M6-04
│   │   │   │       │   │   └── JsonSerializationService.java    # M6-05
│   │   │   │       │   └── mx/
│   │   │   │       │       ├── CircularReferenceResolver.java   # MX-01
│   │   │   │       │       ├── LawVersionMatcher.java           # MX-02
│   │   │   │       │       └── ValidationEngine.java            # MX-03 (107규칙)
│   │   │   │       └── inc/                                     # P 모듈 (개인 전용)
│   │   │   │           ├── p1/
│   │   │   │           │   ├── IncBasicValidator.java           # P1-01
│   │   │   │           │   ├── BusinessPlaceValidator.java      # P1-02
│   │   │   │           │   ├── OtherIncomeValidator.java        # P1-03
│   │   │   │           │   ├── IncDeductionValidator.java       # P1-04
│   │   │   │           │   ├── SincerityValidator.java          # P1-05
│   │   │   │           │   ├── IncForeignTaxValidator.java      # P1-06
│   │   │   │           │   ├── RentalReductionValidator.java    # P1-07
│   │   │   │           │   └── JointBizValidator.java           # P1-08
│   │   │   │           ├── p3/
│   │   │   │           │   ├── IncLocationCheckService.java     # P3-01
│   │   │   │           │   ├── SincerityTargetService.java      # P3-02
│   │   │   │           │   ├── ComprehensiveIncomeService.java  # P3-03
│   │   │   │           │   └── IncTaxBaseService.java           # P3-04
│   │   │   │           └── p4/
│   │   │   │               ├── IncomeDeductionCalc.java         # P4-01
│   │   │   │               ├── ExemptIncomeAllocationCalc.java  # P4-02
│   │   │   │               ├── JointBusinessCalc.java           # P4-03
│   │   │   │               ├── SincerityMedicalEduCalc.java     # P4-04
│   │   │   │               ├── SincerityReportCostCalc.java     # P4-05
│   │   │   │               ├── BookkeepingCreditCalc.java       # P4-06
│   │   │   │               ├── IncForeignTaxCreditCalc.java     # P4-07
│   │   │   │               ├── GoodLandlordCreditCalc.java      # P4-08
│   │   │   │               ├── IncLossCarrybackCalc.java        # P4-09
│   │   │   │               ├── YellowUmbrellaCalc.java          # P4-10
│   │   │   │               ├── PensionSavingsCreditCalc.java    # P4-11
│   │   │   │               ├── ChildCreditCalc.java             # P4-12
│   │   │   │               └── EFilingCreditCalc.java          # P4-13 전자신고 세액공제 (2만원 정액)
│   │   │   │
│   │   │   └── infrastructure/                          # [계층 4] 인프라 계층
│   │   │       ├── persistence/
│   │   │       │   ├── jpa/
│   │   │       │   │   ├── entity/                      # JPA Entity (63개 테이블)
│   │   │       │   │   │   ├── inp/                     # INP_* 입력 테이블
│   │   │       │   │   │   ├── smr/                     # SMR_* 요약 테이블
│   │   │       │   │   │   ├── out/                     # OUT_* 출력 테이블
│   │   │       │   │   │   ├── ref/                     # REF_* 기준정보
│   │   │       │   │   │   └── aud/                     # AUD_* 감사 로그
│   │   │       │   │   └── repository/                  # JpaRepository 구현
│   │   │       │   │       ├── InpRequestJpaRepository.java
│   │   │       │   │       ├── InpRawDatasetJpaRepository.java
│   │   │       │   │       └── ...
│   │   │       │   └── mybatis/
│   │   │       │       ├── mapper/                      # MyBatis Mapper 인터페이스
│   │   │       │       │   ├── SummaryMapper.java
│   │   │       │       │   ├── CreditDetailMapper.java
│   │   │       │       │   ├── CombinationMapper.java
│   │   │       │       │   └── ReportMapper.java
│   │   │       │       └── xml/                         # MyBatis XML 매퍼
│   │   │       │           ├── SummaryMapper.xml
│   │   │       │           ├── CreditDetailMapper.xml
│   │   │       │           └── ReportMapper.xml
│   │   │       ├── config/
│   │   │       │   ├── SecurityConfig.java              # Spring Security 설정
│   │   │       │   ├── JpaConfig.java                   # JPA 설정
│   │   │       │   ├── MyBatisConfig.java               # MyBatis 설정
│   │   │       │   ├── SwaggerConfig.java               # OpenAPI 3.0 설정
│   │   │       │   ├── WebConfig.java                   # CORS, Interceptor 설정
│   │   │       │   ├── JacksonConfig.java               # JSON 직렬화 설정
│   │   │       │   ├── AsyncConfig.java                 # 비동기 처리 설정
│   │   │       │   └── AuditConfig.java                 # JPA Auditing 설정
│   │   │       └── common/
│   │   │           ├── constants/
│   │   │           │   ├── SystemConstants.java          # 시스템 상수
│   │   │           │   ├── TaxConstants.java             # 세무 도메인 상수
│   │   │           │   ├── ErrorCode.java                # 오류 코드 enum
│   │   │           │   ├── MessageCode.java              # UI 메시지 코드 enum
│   │   │           │   └── CategoryCode.java             # 32개 카테고리 코드
│   │   │           ├── exception/
│   │   │           │   ├── BaseException.java            # 기본 예외 (추상)
│   │   │           │   ├── BusinessException.java        # 비즈니스 예외
│   │   │           │   ├── ValidationException.java      # 검증 예외
│   │   │           │   ├── HardFailException.java        # Hard Fail 예외
│   │   │           │   ├── CalculationException.java     # 계산 예외
│   │   │           │   ├── NotFoundException.java        # 리소스 미발견
│   │   │           │   ├── DuplicateRequestException.java # 중복 요청
│   │   │           │   └── TimeoutException.java         # 타임아웃 예외
│   │   │           └── util/
│   │   │               ├── TruncationUtil.java           # 절사 유틸
│   │   │               ├── MoneyCalculator.java          # 금액 계산 유틸
│   │   │               ├── DateTimeUtil.java             # 날짜/시간 유틸
│   │   │               ├── StringUtil.java               # 문자열 유틸
│   │   │               ├── FormatUtil.java               # 포맷 유틸 (금액/한글 등)
│   │   │               ├── MaskingUtil.java              # 마스킹 유틸
│   │   │               ├── CryptoUtil.java               # AES-256 / SHA-256 / Base64
│   │   │               ├── ValidationUtil.java           # Regex 검증 유틸
│   │   │               ├── ChecksumUtil.java             # 체크섬 유틸
│   │   │               ├── ReqIdGenerator.java           # req_id 생성 유틸
│   │   │               ├── TaxYearUtil.java              # 과세연도/기산일 유틸
│   │   │               └── FileUtil.java                 # 파일 업로드/삭제/검증
│   │   │
│   │   └── resources/
│   │       ├── application.yml                          # 메인 설정
│   │       ├── application-local.yml                    # 로컬 프로파일
│   │       ├── application-dev.yml                      # 개발 프로파일
│   │       ├── application-prod.yml                     # 운영 프로파일
│   │       ├── logback-spring.xml                       # 로깅 설정
│   │       ├── messages/
│   │       │   ├── messages_ko.properties               # UI 메시지 (한국어)
│   │       │   ├── errors_ko.properties                 # 오류 메시지 (한국어)
│   │       │   └── validation_ko.properties             # 검증 메시지 (한국어)
│   │       ├── mybatis/
│   │       │   ├── mybatis-config.xml                   # MyBatis 전역 설정
│   │       │   └── mapper/                              # XML 매퍼
│   │       ├── db/
│   │       │   ├── migration/                           # Flyway 마이그레이션
│   │       │   │   ├── V1__create_inp_tables.sql
│   │       │   │   ├── V2__create_smr_tables.sql
│   │       │   │   ├── V3__create_out_tables.sql
│   │       │   │   ├── V4__create_ref_tables.sql
│   │       │   │   ├── V5__create_aud_tables.sql
│   │       │   │   ├── V6__create_indexes.sql
│   │       │   │   ├── V7__create_triggers.sql
│   │       │   │   └── V8__seed_reference_data.sql
│   │       │   └── seed/
│   │       │       ├── ref_tax_rate_bracket.sql          # 2018~2025 세율
│   │       │       ├── ref_min_tax_rate.sql              # 최저한세율
│   │       │       ├── ref_mutual_exclusion.sql          # 상호배제 10규칙
│   │       │       └── ref_industry_eligibility.sql      # 업종 적격성
│   │       └── static/
│   │
│   └── test/
│       ├── java/com/taxservice/entec/
│       │   ├── domain/service/module/m4/                # 57개 공식 단위 테스트
│       │   │   ├── InvestmentCreditCalcTest.java
│       │   │   ├── EmploymentCreditCalcTest.java
│       │   │   └── ...
│       │   ├── infrastructure/common/util/              # 유틸 테스트
│       │   │   ├── TruncationUtilTest.java
│       │   │   └── MoneyCalculatorTest.java
│       │   └── integration/                             # 통합 테스트
│       │       ├── CorpE2ETest.java                     # 법인 E2E
│       │       └── IncE2ETest.java                      # 개인 E2E
│       └── resources/
│           └── test-data/                               # 테스트 데이터
│               ├── corp_basic_sample.json
│               └── inc_basic_sample.json
```

---

## 3. pom.xml 의존성 설계

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.5</version>
    </parent>

    <groupId>com.taxservice</groupId>
    <artifactId>tax-service-entec</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>TaxServiceENTEC</name>
    <description>통합 경정청구 환급액 산출 시스템</description>

    <properties>
        <java.version>17</java.version>
        <mybatis-spring-boot.version>3.0.3</mybatis-spring-boot.version>
        <springdoc.version>2.3.0</springdoc.version>
        <jjwt.version>0.12.5</jjwt.version>
        <flyway.version>9.22.3</flyway.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Core -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- JPA (Hibernate) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- MyBatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis-spring-boot.version}</version>
        </dependency>

        <!-- PostgreSQL -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Flyway DB Migration -->
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>

        <!-- Spring Security + JWT -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>${jjwt.version}</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>${jjwt.version}</version>
            <scope>runtime</scope>
        </dependency>

        <!-- OpenAPI (Swagger) -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>${springdoc.version}</version>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- Jackson (JSONB 지원) -->
        <dependency>
            <groupId>com.vladmihalcea</groupId>
            <artifactId>hibernate-types-60</artifactId>
            <version>2.21.1</version>
        </dependency>

        <!-- AOP (감사 로그) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <!-- Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

---

## 4. Spring Boot Application 및 설정 파일

### 4.1 TaxServiceEntecApplication.java

```java
/**
 * 통합 경정청구 환급액 산출 시스템 - Spring Boot Main Application.
 *
 * <p>법인(CORP)과 개인사업자(INC) 경정청구 환급액 극대화를 위한
 * AI 기반 통합 점검 시스템의 진입점.</p>
 *
 * @author TaxServiceENTEC
 * @version 1.0.0
 * @since 2026-02-18
 */
@SpringBootApplication
@EnableJpaAuditing
public class TaxServiceEntecApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaxServiceEntecApplication.class, args);
    }
}
```

### 4.2 application.yml (메인)

```yaml
spring:
  application:
    name: tax-service-entec
  profiles:
    active: local

  # JPA 공통 설정
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: true
        default_batch_fetch_size: 100
        jdbc:
          batch_size: 50

  # MyBatis 공통 설정
  mybatis:
    config-location: classpath:mybatis/mybatis-config.xml
    mapper-locations: classpath:mybatis/mapper/**/*.xml
    type-aliases-package: com.taxservice.entec.domain.model.entity

  # Jackson 설정
  jackson:
    serialization:
      write-dates-as-timestamps: false
      fail-on-empty-beans: false
    deserialization:
      fail-on-unknown-properties: false
    default-property-inclusion: non_null
    date-format: yyyy-MM-dd'T'HH:mm:ssXXX
    time-zone: Asia/Seoul

  # Flyway 설정
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

# 애플리케이션 커스텀 설정
app:
  calculation:
    timeout-ms: 30000                    # 계산 타임아웃 30초
    bb-max-items: 15                      # Branch & Bound 임계값
    convergence-max-iter: 5               # 순환참조 최대 반복
    convergence-tolerance: 1              # 수렴 판단 기준 (원)
  security:
    jwt:
      secret-key: ${JWT_SECRET_KEY}
      expiration-ms: 28800000             # 8시간
    crypto:
      aes-secret-key: ${AES_SECRET_KEY}
  data:
    max-category-count: 40                # 최대 카테고리 수/요청
    max-category-size-mb: 10              # 카테고리별 최대 크기
    max-payload-size-mb: 50               # 전체 페이로드 최대 크기
    max-employee-rows: 10000              # employee_detail 최대 행 수
    retention-days: 90                    # 데이터 보관 기간
  request:
    idempotency-ttl-hours: 24             # 멱등성 키 캐시 TTL

# 로깅 설정
logging:
  level:
    com.taxservice.entec: INFO
    org.hibernate.SQL: DEBUG
    org.mybatis: DEBUG
```

### 4.3 application-local.yml

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/taxservice_entec
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:postgres}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000

  jpa:
    show-sql: true
    hibernate:
      ddl-auto: validate

logging:
  level:
    com.taxservice.entec: DEBUG
```

### 4.4 application-prod.yml

```yaml
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL}
    username: ${SPRING_DATASOURCE_USERNAME}
    password: ${SPRING_DATASOURCE_PASSWORD}
    hikari:
      maximum-pool-size: 30
      minimum-idle: 10
      connection-timeout: 20000

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: none

logging:
  level:
    com.taxservice.entec: INFO
    org.hibernate.SQL: WARN
```

---

## 5. Config 파일 설계

### 5.1 SecurityConfig.java

```java
/**
 * Spring Security 설정.
 *
 * <p>JWT 기반 인증과 RBAC 4단계(ADMIN/MANAGER/ACCOUNTANT/VIEWER)
 * 권한 체계를 정의한다.</p>
 *
 * <p>주요 보안 정책:
 * <ul>
 *   <li>API-01 (POST /requests): ACCOUNTANT 이상 접근</li>
 *   <li>API-02 (POST /analyze): ACCOUNTANT 이상 접근</li>
 *   <li>API-07 (GET /reference): VIEWER 이상 접근</li>
 *   <li>API-11 (GET /audit-log): ADMIN만 접근</li>
 * </ul></p>
 */
@Configuration
@EnableWebSecurity
public class SecurityConfig { ... }
```

### 5.2 JpaConfig.java

```java
/**
 * JPA 설정 및 Auditing.
 *
 * <p>JPA 엔티티 스캔 범위, 트랜잭션 매니저, created_at/updated_at
 * 자동 설정을 정의한다.</p>
 *
 * <p>PostgreSQL 15의 JSONB 컬럼 타입 매핑을 위해
 * hibernate-types-60 라이브러리의 JsonBinaryType을 활성화한다.</p>
 */
@Configuration
@EnableJpaRepositories(basePackages = "com.taxservice.entec.infrastructure.persistence.jpa.repository")
@EntityScan(basePackages = "com.taxservice.entec.infrastructure.persistence.jpa.entity")
public class JpaConfig { ... }
```

### 5.3 MyBatisConfig.java

```java
/**
 * MyBatis 설정.
 *
 * <p>복잡 집계 쿼리, 동적 SQL이 필요한 보고서 조회,
 * 조합 비교 쿼리에 MyBatis를 사용한다.</p>
 *
 * <p>JPA와 동일한 DataSource를 공유하며,
 * SqlSessionFactory에 JSONB 타입 핸들러를 등록한다.</p>
 */
@Configuration
@MapperScan(basePackages = "com.taxservice.entec.infrastructure.persistence.mybatis.mapper")
public class MyBatisConfig { ... }
```

### 5.4 기타 Config 파일

| Config 클래스 | 역할 |
|--------------|------|
| **SwaggerConfig** | OpenAPI 3.0 문서 설정, API 그룹 분리 (요청/분석/보고서/기준정보/감사) |
| **WebConfig** | CORS 정책, ReqIdInterceptor 등록, 요청 로깅 Interceptor |
| **JacksonConfig** | BigDecimal 직렬화 정책, JSONB 변환기, 날짜 포맷 통일 |
| **AsyncConfig** | 비동기 처리 ThreadPool 설정 (employee_detail 10,000건 초과 시) |
| **AuditConfig** | JPA Auditing AuditorAware 설정 (현재 인증 사용자) |

---

## 6. 공통 엔티티/DTO 설계

### 6.1 Value Object

#### ReqId.java — 요청 ID Value Object

```java
/**
 * 경정청구 요청 ID Value Object.
 *
 * <p>형식: {C/I}-{사업자번호10}-{날짜8}-{일련번호3}</p>
 * <p>예: C-1234567890-20260218-001</p>
 *
 * <p>불변 객체로, 생성 시 형식 검증을 수행한다.
 * req_id는 시스템 전 계층을 관통하는 유일 키이다.</p>
 *
 * @see ReqIdGenerator 생성 유틸
 */
@Embeddable
public class ReqId {
    private static final Pattern PATTERN =
        Pattern.compile("^[CI]-\\d{10}-\\d{8}-\\d{3}$");

    @Column(name = "req_id", length = 30, nullable = false)
    private String value;

    /** 세목 추출 (C→CORP, I→INC) */
    public TaxType getTaxType() { ... }
    /** 사업자번호 추출 */
    public String getTaxpayerId() { ... }
    /** 요청일자 추출 */
    public LocalDate getRequestDate() { ... }
}
```

#### MoneyAmount.java — 금액 Value Object

```java
/**
 * 금액 Value Object (절사 정책 내장).
 *
 * <p>모든 금액 계산은 이 클래스를 통해 수행하여
 * 10원 미만 TRUNCATE 정책을 보장한다.</p>
 *
 * <p>사용 예:
 * <pre>
 *   MoneyAmount credit = MoneyAmount.of(15678923);  // 15,678,920 (절사)
 *   MoneyAmount result = credit.multiply(0.10);      // 1,567,890 (절사)
 * </pre></p>
 *
 * @see TruncationUtil 절사 유틸리티
 */
@Embeddable
public class MoneyAmount implements Comparable<MoneyAmount> {
    /** 원 단위 금액 (10원 미만 절사 완료) */
    private long value;

    public static MoneyAmount of(long rawAmount) { ... }
    public MoneyAmount add(MoneyAmount other) { ... }
    public MoneyAmount subtract(MoneyAmount other) { ... }
    public MoneyAmount multiply(double rate) { ... }
    public boolean isPositive() { ... }
    public boolean isZeroOrNegative() { ... }
}
```

#### TaxRate.java — 세율 Value Object

```java
/**
 * 세율 Value Object (소수점 3자리 절사).
 *
 * <p>모든 세율/공제율은 이 클래스를 통해 관리한다.
 * 소수점 4자리째에서 절사(TRUNCATE)하여 3자리만 유효하다.</p>
 *
 * @param value 세율 값 (예: 0.100 = 10%)
 */
@Embeddable
public class TaxRate {
    private double value;

    public static TaxRate of(double rawRate) { ... }
    public MoneyAmount applyTo(MoneyAmount base) { ... }
}
```

### 6.2 핵심 엔티티 (JPA)

#### InpRequest.java (INP_REQUEST 테이블)

```java
/**
 * 경정청구 요청 마스터 엔티티.
 *
 * <p>1건의 경정청구 진단 작업 단위를 나타내며,
 * req_id가 시스템 전 계층을 관통하는 유일 키이다.</p>
 *
 * <p>상태 전이: received → parsing → parsed → preparing →
 * checking → calculating → optimizing → reporting → completed / error</p>
 *
 * @see RequestStatus 상태 enum
 * @see InpRawDataset JSON 원본 데이터
 */
@Entity
@Table(name = "inp_request")
public class InpRequest extends BaseTimeEntity {
    @Id
    @Column(name = "req_id", length = 30)
    private String reqId;

    @Column(name = "applicant_id", length = 20, nullable = false)
    private String applicantId;

    @Column(name = "taxpayer_id", length = 20, nullable = false)
    private String taxpayerId;

    @Enumerated(EnumType.STRING)
    @Column(name = "tax_type", length = 4, nullable = false)
    private TaxType taxType;

    @Column(name = "tax_year", nullable = false)
    private Short taxYear;

    @Enumerated(EnumType.STRING)
    @Column(name = "request_status", length = 20, nullable = false)
    private RequestStatus requestStatus;

    @Column(name = "idempotency_key", length = 36)
    private String idempotencyKey;
    // ... 나머지 컬럼 (schema.md 참조)
}
```

#### InpRawDataset.java (INP_RAW_DATASET 테이블)

```java
/**
 * JSON 원본 데이터셋 보관 엔티티.
 *
 * <p>세부 입력자료를 JSON 형태의 데이터셋으로 받아서
 * 원본 그대로 보관한다. INSERT ONLY 정책으로
 * UPDATE/DELETE는 DB 트리거로 차단된다.</p>
 *
 * <p>32개 카테고리: 공통(8) + CORP(16) + INC(8)</p>
 *
 * @see CategoryCode 카테고리 코드 enum
 */
@Entity
@Table(name = "inp_raw_dataset")
@TypeDef(name = "jsonb", typeClass = JsonBinaryType.class)
public class InpRawDataset {
    @Id
    @Column(name = "req_id", length = 30)
    private String reqId;

    @Id
    @Column(name = "category", length = 30)
    private String category;

    @Type(type = "jsonb")
    @Column(name = "raw_json", columnDefinition = "jsonb", nullable = false)
    private Map<String, Object> rawJson;

    @Column(name = "schema_version", length = 10, nullable = false)
    private String schemaVersion;

    @Column(name = "checksum", length = 64)
    private String checksum;
    // ...
}
```

### 6.3 공통 DTO

#### AmendmentClaimRequest.java (API-01 요청)

```java
/**
 * 경정청구 요청 접수 DTO.
 *
 * <p>외부에서 JSON 형태로 전달받는 경정청구 요청 데이터.
 * datasets 배열에 카테고리별 입력 데이터셋을 포함한다.</p>
 *
 * @param applicantType 신청인 구분: C(법인), I(개인)
 * @param applicantId   신청인 ID (세무사/법인)
 * @param taxType       세목: CORP / INC
 * @param taxYear       경정청구 대상 과세연도
 * @param datasets      카테고리별 입력 데이터셋 목록
 */
public class AmendmentClaimRequest {
    @NotBlank
    private String applicantType;

    @NotBlank
    private String applicantId;

    @NotNull
    private TaxType taxType;

    @NotNull
    @Min(2018) @Max(2025)
    private Integer taxYear;

    @NotEmpty
    @Size(max = 40)
    private List<DatasetItem> datasets;
}
```

#### DatasetItem.java

```java
/**
 * 개별 데이터셋 항목 DTO.
 *
 * @param category      카테고리 코드 (32개 중 1개)
 * @param schemaVersion JSON 스키마 버전
 * @param data          원본 JSON 데이터
 */
public class DatasetItem {
    @NotBlank
    private String category;

    @NotBlank
    private String schemaVersion;

    @NotNull
    private Object data;
}
```

#### ErrorResponse.java (표준 오류 응답)

```java
/**
 * 표준 오류 응답 DTO.
 *
 * <p>모든 API 오류는 이 형식으로 통일 응답한다.</p>
 *
 * @param code      오류 코드 (ErrorCode enum)
 * @param message   사용자 표시용 메시지
 * @param details   검증 오류 상세 (필드별)
 * @param reqId     관련 요청 ID (있는 경우)
 * @param timestamp 오류 발생 시각
 * @param traceId   추적 ID
 */
public class ErrorResponse {
    private String code;
    private String message;
    private List<FieldError> details;
    private String reqId;
    private String timestamp;
    private String traceId;
}
```

### 6.4 BaseTimeEntity (JPA Auditing 공통)

```java
/**
 * JPA Auditing 공통 엔티티.
 *
 * <p>모든 엔티티의 생성일시/수정일시를 자동 관리한다.</p>
 */
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

---

## 7. 예외 처리 체계

### 7.1 예외 계층 구조

```
BaseException (abstract)
├── BusinessException            -- 비즈니스 규칙 위반 (HTTP 400/422)
│   ├── ValidationException      -- 입력 데이터 검증 실패 (400)
│   ├── HardFailException        -- 경정청구 불가 사유 (422)
│   └── CalculationException     -- 계산 중 오류 (422)
├── NotFoundException            -- 리소스 미발견 (404)
├── DuplicateRequestException    -- 중복 요청 / 상태 충돌 (409)
├── TimeoutException             -- 계산 타임아웃 (504)
└── SystemException              -- 시스템 내부 오류 (500)
```

### 7.2 BaseException

```java
/**
 * 시스템 기본 예외 (추상 클래스).
 *
 * <p>모든 커스텀 예외의 루트. ErrorCode와 HTTP 상태를 내장하여
 * GlobalExceptionHandler에서 통일 처리한다.</p>
 *
 * @param errorCode 오류 코드 (ErrorCode enum)
 * @param message   상세 메시지
 * @param reqId     관련 req_id (nullable)
 */
public abstract class BaseException extends RuntimeException {
    private final ErrorCode errorCode;
    private final String reqId;

    public abstract int getHttpStatus();
}
```

### 7.3 ErrorCode enum

```java
/**
 * 시스템 오류 코드 정의.
 *
 * <p>형식: ERR_{카테고리}_{상세}</p>
 * <p>각 코드는 HTTP 상태 코드와 메시지 키를 내장한다.</p>
 */
public enum ErrorCode {
    // 입력 검증 (400)
    ERR_INVALID_JSON(400, "error.invalid.json"),
    ERR_VALIDATION_FAILED(400, "error.validation.failed"),
    ERR_MISSING_REQUIRED_CATEGORY(400, "error.missing.required.category"),
    ERR_INVALID_TAX_TYPE(400, "error.invalid.tax.type"),
    ERR_INVALID_REQ_ID_FORMAT(400, "error.invalid.req.id.format"),
    ERR_PAYLOAD_TOO_LARGE(400, "error.payload.too.large"),

    // 비즈니스 (422)
    ERR_HARD_FAIL(422, "error.hard.fail"),
    ERR_CALCULATION_FAILED(422, "error.calculation.failed"),
    ERR_CONVERGENCE_FAILED(422, "error.convergence.failed"),
    ERR_DEADLINE_EXPIRED(422, "error.deadline.expired"),

    // 리소스 (404)
    ERR_REQUEST_NOT_FOUND(404, "error.request.not.found"),
    ERR_REPORT_NOT_FOUND(404, "error.report.not.found"),

    // 상태 충돌 (409)
    ERR_INVALID_STATUS(409, "error.invalid.status"),
    ERR_DUPLICATE_REQUEST(409, "error.duplicate.request"),

    // 타임아웃 (504)
    ERR_TIMEOUT(504, "error.timeout"),

    // 시스템 (500)
    ERR_INTERNAL(500, "error.internal"),
    ERR_DATABASE(500, "error.database");

    private final int httpStatus;
    private final String messageKey;
}
```

### 7.4 GlobalExceptionHandler

```java
/**
 * 통합 예외 처리 Handler.
 *
 * <p>모든 예외를 ErrorResponse 형태로 통일 응답한다.
 * 비즈니스 예외는 상세 메시지를 포함하고,
 * 시스템 예외는 내부 정보를 숨긴다.</p>
 *
 * <p>모든 예외에 대해 req_id 기반 감사 로그를 기록한다.</p>
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    /**
     * BaseException 처리.
     *
     * @param ex 발생한 예외
     * @return ErrorResponse (오류코드, 메시지, req_id 포함)
     */
    @ExceptionHandler(BaseException.class)
    public ResponseEntity<ErrorResponse> handleBaseException(BaseException ex) { ... }

    /**
     * Bean Validation 예외 처리.
     *
     * @param ex MethodArgumentNotValidException
     * @return ErrorResponse (필드별 오류 상세 포함)
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException ex) { ... }

    /**
     * 예상치 못한 시스템 예외 처리.
     * 내부 스택트레이스는 로그에만 기록하고 클라이언트에게 노출하지 않는다.
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleUnexpected(Exception ex) { ... }
}
```

---

## 8. 공통 요소 및 유틸리티

### 8.1 공통 상수/코드 (Constants & Global Codes)

#### SystemConstants.java

```java
/**
 * 시스템 전반 상수 정의.
 *
 * <p>HTTP 상태, 시스템 제한값, 기본 설정값 등
 * 시스템 전반에서 참조하는 고정값.</p>
 */
public final class SystemConstants {
    private SystemConstants() {} // 인스턴스화 방지

    // === HTTP 상태 코드 ===
    public static final int HTTP_OK = 200;
    public static final int HTTP_CREATED = 201;
    public static final int HTTP_BAD_REQUEST = 400;
    public static final int HTTP_NOT_FOUND = 404;
    public static final int HTTP_CONFLICT = 409;
    public static final int HTTP_UNPROCESSABLE = 422;
    public static final int HTTP_INTERNAL = 500;
    public static final int HTTP_TIMEOUT = 504;

    // === 시스템 제한값 ===
    public static final int MAX_CATEGORY_COUNT = 40;
    public static final long MAX_CATEGORY_SIZE_BYTES = 10L * 1024 * 1024;
    public static final long MAX_PAYLOAD_SIZE_BYTES = 50L * 1024 * 1024;
    public static final int MAX_EMPLOYEE_ROWS = 10_000;
    public static final int MAX_RETRY_COUNT = 3;
    public static final long RETRY_BASE_MS = 100L;

    // === 기본값 ===
    public static final int DEFAULT_BB_THRESHOLD = 15;
    public static final int DEFAULT_MAX_CONVERGENCE = 5;
    public static final int DEFAULT_CONVERGENCE_TOLERANCE = 1;
    public static final int DATA_RETENTION_DAYS = 90;
    public static final String DEFAULT_SCHEMA_VERSION = "1.0";
}
```

#### TaxConstants.java

```java
/**
 * 세무 도메인 상수 정의.
 *
 * <p>세법에 기반한 고정 코드, 기본값, 연도 상수 등
 * 환급액 계산에서 참조하는 세무 도메인 전용 상수.</p>
 */
public final class TaxConstants {
    private TaxConstants() {}

    // === 세목 코드 ===
    public static final String TAX_TYPE_CORP = "CORP";
    public static final String TAX_TYPE_INC = "INC";

    // === 조항 코드 (Provision Code) ===
    public static final String PROV_INVESTMENT = "SS24";         // 통합투자
    public static final String PROV_EMPLOYMENT = "SS29_8";       // 통합고용
    public static final String PROV_SOCIAL_INSURANCE = "SS30_4"; // 사회보험료
    public static final String PROV_STARTUP = "SS6";             // 창업감면
    public static final String PROV_SME_SPECIAL = "SS7";         // 중소특별
    public static final String PROV_RD = "SS10";                 // R&D
    public static final String PROV_FOREIGN_TAX_CORP = "LAW57C"; // 법인 외국납부
    public static final String PROV_LOSS_CF = "LAW13";           // 이월결손금

    // === 기업 규모 ===
    public static final String SIZE_SMALL = "SMALL";
    public static final String SIZE_MEDIUM = "MEDIUM";
    public static final String SIZE_MID_LARGE = "MID_LARGE";
    public static final String SIZE_LARGE = "LARGE";

    // === 경정청구 기한 ===
    public static final int AMENDMENT_CLAIM_YEARS = 5;

    // === 이월공제 기한 ===
    public static final int CARRYFORWARD_YEARS = 10;
    public static final int LOSS_CF_YEARS_NEW = 15; // 2020.1.1 이후 발생
    public static final int LOSS_CF_YEARS_OLD = 10; // 2020.1.1 이전 발생

    // === 절사 단위 ===
    public static final int TRUNCATE_AMOUNT_UNIT = 10;           // 금액 10원
    public static final int TRUNCATE_RATE_DECIMALS = 3;          // 비율 소수점 3자리
    public static final int TRUNCATE_EMPLOYEE_DECIMALS = 2;      // 인원 소수점 2자리
    public static final int TRUNCATE_INTEREST_UNIT = 1;          // 환급가산금 1원

    // === 감면한도 ===
    public static final long SME_SPECIAL_LIMIT = 100_000_000L;   // 중소특별 1억원 한도

    // === 농특세율 ===
    public static final double AGRICULTURAL_TAX_RATE = 0.20;

    // === 개인 최저한세 기준 (조특법 §132) ===
    public static final long INC_MIN_TAX_THRESHOLD = 30_000_000L;
    public static final double INC_MIN_TAX_RATE_LOW = 0.35;      // 3천만 이하
    public static final double INC_MIN_TAX_RATE_HIGH = 0.45;     // 3천만 초과
}
```

#### CategoryCode.java

```java
/**
 * 입력 데이터셋 32개 카테고리 코드 enum.
 *
 * <p>공통(8) + CORP전용(16) + INC전용(8) = 32개.
 * 각 카테고리는 세목 범위(scope)와 필수 여부(required)를 내장한다.</p>
 */
public enum CategoryCode {
    // === 공통 (8개) ===
    EMPLOYEE_DETAIL("employee_detail", Scope.BOTH, false),
    EMPLOYEE_MONTHLY("employee_monthly", Scope.BOTH, false),
    INVESTMENT("investment", Scope.BOTH, false),
    STARTUP("startup", Scope.BOTH, false),
    SME_SPECIAL("sme_special", Scope.BOTH, false),
    RD_EXPENSE("rd_expense", Scope.BOTH, false),
    EXISTING_DEDUCTION("existing_deduction", Scope.BOTH, false),
    FOREIGN_TAX("foreign_tax", Scope.BOTH, false),

    // === CORP 전용 (16개) ===
    CORP_BASIC("corp_basic", Scope.CORP, true),  // CORP 필수
    REPRESENTATIVE("representative", Scope.CORP, false),
    LOSS_CARRYFORWARD("loss_carryforward", Scope.CORP, false),
    // ... (16개)

    // === INC 전용 (8개) ===
    INC_BASIC("inc_basic", Scope.INC, true),     // INC 필수
    INC_BUSINESS("inc_business", Scope.INC, false),
    // ... (8개)
    ;

    private final String code;
    private final Scope scope;
    private final boolean required;

    public enum Scope { CORP, INC, BOTH }
}
```

### 8.2 공통 함수 (Common Functions)

| 함수/서비스 | 역할 | 위치 |
|------------|------|------|
| **AuditLogService** | 계산 이력 APPEND ONLY 기록 | `application/service/` |
| **TransactionTemplate** | 프로그래밍 방식 트랜잭션 (TX-1, TX-2) | Spring 내장 |
| **ErrorHandlingAspect** | AOP 기반 에러 로깅 + req_id 추적 | `infrastructure/common/` |
| **RequestContextHolder** | 현재 요청의 req_id, applicant_id 보관 (ThreadLocal) | `infrastructure/common/` |
| **RetryTemplate** | 지수 백오프 재시도 (req_id 발급 동시성) | `infrastructure/common/` |

### 8.3 유틸리티 함수 (Utility Functions)

#### TruncationUtil.java

```java
/**
 * 절사(Truncation) 유틸리티.
 *
 * <p>국세기본법 §86에 따른 금액 절사, 비율 절사, 인원 절사를 수행한다.
 * 반올림은 절대 금지이며, 모든 금액 처리에 이 유틸을 사용한다.</p>
 */
public final class TruncationUtil {
    private TruncationUtil() {}

    /**
     * 금액 절사 (10원 미만 TRUNCATE).
     *
     * @param amount 원본 금액 (원)
     * @return 10원 미만 절사된 금액
     */
    public static long truncateAmount(long amount) {
        return (amount / 10) * 10;
    }

    /**
     * 비율 절사 (소수점 3자리 TRUNCATE).
     *
     * @param rate 원본 비율 (예: 0.12345)
     * @return 소수점 4자리째 절사된 비율 (예: 0.123)
     */
    public static double truncateRate(double rate) {
        return Math.floor(rate * 1000) / 1000;
    }

    /**
     * 상시근로자 수 절사 (소수점 2자리, 3자리째 절사).
     *
     * @param count 원본 인원 수
     * @return 소수점 3자리째 절사된 인원 수
     */
    public static double truncateEmployee(double count) {
        return Math.floor(count * 100) / 100;
    }

    /**
     * 환급가산금 절사 (1원 미만 TRUNCATE).
     *
     * @param interest 원본 가산금 (원)
     * @return 1원 미만 절사된 가산금
     */
    public static long truncateInterest(long interest) {
        return interest; // long 타입이므로 자동 절사
    }
}
```

#### CryptoUtil.java

```java
/**
 * 암호화/해시 유틸리티.
 *
 * <p>개인정보(주민번호, 사업자번호) 암호화 및 데이터 무결성 검증.</p>
 * <ul>
 *   <li>AES-256: 주민번호/사업자번호 암복호화</li>
 *   <li>SHA-256: JSON 데이터 체크섬 생성</li>
 *   <li>Base64: 인코딩/디코딩</li>
 * </ul>
 */
public final class CryptoUtil {
    private CryptoUtil() {}

    /**
     * AES-256 암호화.
     *
     * @param plainText 원본 텍스트
     * @param secretKey 암호화 키 (32바이트)
     * @return Base64 인코딩된 암호문
     * @throws CryptoException 암호화 실패 시
     */
    public static String encryptAes256(String plainText, String secretKey) { ... }

    /**
     * AES-256 복호화.
     *
     * @param cipherText Base64 인코딩된 암호문
     * @param secretKey  암호화 키 (32바이트)
     * @return 복호화된 원본 텍스트
     * @throws CryptoException 복호화 실패 시
     */
    public static String decryptAes256(String cipherText, String secretKey) { ... }

    /**
     * SHA-256 해시 생성.
     *
     * @param data 원본 데이터 (바이트 배열)
     * @return 16진수 해시 문자열 (64자)
     */
    public static String sha256(byte[] data) { ... }

    /**
     * Base64 인코딩.
     *
     * @param data 원본 데이터
     * @return Base64 인코딩 문자열
     */
    public static String encodeBase64(byte[] data) { ... }

    /**
     * Base64 디코딩.
     *
     * @param encoded Base64 문자열
     * @return 디코딩된 바이트 배열
     */
    public static byte[] decodeBase64(String encoded) { ... }
}
```

#### 기타 유틸리티 목록

| 유틸리티 | 역할 | 주요 메서드 |
|---------|------|-----------|
| **DateTimeUtil** | 날짜/시간/과세연도/경정청구기한 계산 | `getClaimDeadline(taxYear, taxType)`, `getFiscalYearEnd()`, `calcDaysBetween()` |
| **StringUtil** | 문자열 조작 (null-safe) | `isEmpty()`, `leftPad()`, `truncate()` |
| **FormatUtil** | 금액 포맷/한글 변환/사업자번호 포맷 | `formatMoney(123456789)` → "123,456,789원", `toKoreanMoney()` → "일억이천..." |
| **MaskingUtil** | 개인정보 마스킹 | `maskBizRegNo("123-45-67890")` → "123-45-6****" |
| **ValidationUtil** | Regex 검증/사업자번호 체크 | `isValidBizRegNo()`, `isValidReqId()`, `isValidKSIC()` |
| **ChecksumUtil** | 데이터 무결성 체크섬 | `generateChecksum(jsonBytes)` → SHA-256 해시 |
| **ReqIdGenerator** | req_id 생성 | `generate(applicantType, bizRegNo)` → "C-1234567890-20260218-001" |
| **TaxYearUtil** | 과세연도 관련 | `getEffectiveLawYear()`, `isWithinClaimDeadline()`, `getInterestStartDate()` |
| **FileUtil** | 파일 업로드/삭제/검증 | `validateFileSize()`, `saveToStorage()`, `deleteFile()` |
| **MoneyCalculator** | BigDecimal 기반 정밀 계산 | `multiply()`, `divide()`, `applyProgressiveTax()` |

---

## 9. 접근 권한 / 로깅 / 감사 기능

### 9.1 RBAC 권한 체계

| 역할 | 코드 | 권한 범위 |
|------|------|----------|
| **시스템 관리자** | ADMIN | 전체 접근, 기준정보 관리, 감사 로그 조회 |
| **세무 관리자** | MANAGER | 요청 조회/실행, 보고서 조회, 기준정보 조회 |
| **세무사/담당자** | ACCOUNTANT | 본인 요청 생성/조회, 보고서 조회 |
| **조회자** | VIEWER | 본인 관련 결과 조회만 |

### 9.2 접근 권한 규칙

```java
/**
 * 접근 권한 검증 서비스.
 *
 * <p>신청인(applicant) 본인의 요청만 접근 가능하며,
 * ADMIN/MANAGER는 모든 요청에 접근할 수 있다.</p>
 *
 * <p>규칙:
 * <ul>
 *   <li>ADMIN: 모든 req_id 접근 가능</li>
 *   <li>MANAGER: 모든 req_id 접근 가능</li>
 *   <li>ACCOUNTANT: 본인 applicant_id의 req_id만 접근</li>
 *   <li>VIEWER: 본인 applicant_id의 req_id만 읽기 접근</li>
 * </ul></p>
 */
@Service
public class AccessControlService {
    /**
     * 요청에 대한 접근 권한을 확인한다.
     *
     * @param reqId         대상 요청 ID
     * @param currentUser   현재 인증된 사용자
     * @throws AccessDeniedException 접근 권한 없을 시
     */
    public void verifyAccess(String reqId, AuthenticatedUser currentUser) { ... }
}
```

### 9.3 감사 로그 (AUD_CALCULATION_LOG)

```java
/**
 * 감사 로그 서비스.
 *
 * <p>모든 계산 이력 및 접근 이력을 APPEND ONLY로 기록한다.
 * AOP(@Auditable 어노테이션)로 자동 기록 또는
 * 명시적 호출로 기록할 수 있다.</p>
 *
 * <p>기록 대상:
 * <ul>
 *   <li>모듈별 계산 입력/출력 스냅샷 (JSON)</li>
 *   <li>실행 시간 (ms)</li>
 *   <li>성공/실패 상태</li>
 *   <li>오류 메시지 (실패 시)</li>
 *   <li>접근 이력 (req_id 조회 등)</li>
 * </ul></p>
 */
@Service
public class AuditLogService {
    /**
     * 계산 모듈 실행 이력을 기록한다.
     *
     * @param reqId           요청 ID
     * @param moduleCode      모듈 코드 (M1-01, M4-01 등)
     * @param stepName        단계명
     * @param inputSnapshot   입력 스냅샷 (JSON)
     * @param outputSnapshot  출력 스냅샷 (JSON)
     * @param executionTimeMs 실행 시간 (ms)
     * @param status          SUCCESS / ERROR / WARNING
     * @param errorMessage    오류 메시지 (nullable)
     */
    public void logCalculation(String reqId, String moduleCode, String stepName,
                               Object inputSnapshot, Object outputSnapshot,
                               int executionTimeMs, String status, String errorMessage) { ... }
}
```

### 9.4 요청 추적 로깅

```java
/**
 * 요청 추적 Interceptor.
 *
 * <p>모든 API 요청에 대해 req_id (또는 trace_id)를 MDC에 설정하여
 * Logback 로그에 자동 포함시킨다.</p>
 *
 * <p>MDC 키:
 * <ul>
 *   <li>req_id: 경정청구 요청 ID</li>
 *   <li>trace_id: API 호출 추적 ID (UUID)</li>
 *   <li>user_id: 인증된 사용자 ID</li>
 * </ul></p>
 */
@Component
public class RequestTraceInterceptor implements HandlerInterceptor { ... }
```

**logback-spring.xml 패턴**:
```xml
<pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%thread] [%X{trace_id}] [%X{req_id}]
         [%X{user_id}] %-5level %logger{36} - %msg%n</pattern>
```

---

## 10. UI 메시지 / 오류 메시지 / 사용자 메시지

### 10.1 메시지 파일 구조

#### messages_ko.properties (UI 알림 메시지)

```properties
# === 요청 관련 ===
msg.request.created=경정청구 요청이 접수되었습니다. 요청번호: {0}
msg.request.analyzing=점검을 실행 중입니다. 잠시 기다려 주십시오.
msg.request.completed=점검이 완료되었습니다. 보고서를 확인해 주십시오.
msg.request.status.received=요청 접수 완료
msg.request.status.parsing=입력 데이터 파싱 중
msg.request.status.parsed=입력 데이터 파싱 완료
msg.request.status.preparing=사전 데이터 준비 중
msg.request.status.checking=사전 점검 수행 중
msg.request.status.calculating=공제/감면 산출 중
msg.request.status.optimizing=최적 조합 탐색 중
msg.request.status.reporting=보고서 생성 중
msg.request.status.completed=처리 완료
msg.request.status.error=처리 중 오류 발생

# === 보고서 관련 ===
msg.report.generated=환급 분석 보고서가 생성되었습니다.
msg.report.refund.amount=예상 환급액: {0}원
msg.report.no.refund=추가 환급 가능 금액이 없습니다.
msg.report.warning=일부 항목에 확인이 필요합니다. 상세 내역을 확인해 주십시오.
msg.report.disclaimer=본 산출결과는 참고용이며, 최종 환급액은 과세관청의 결정에 따릅니다.

# === 계산 관련 ===
msg.calc.convergence.success=순환참조가 {0}회 반복 후 수렴하였습니다.
msg.calc.convergence.warning=순환참조가 {0}회 내 수렴하지 못했습니다. 최선 결과를 적용합니다.
msg.calc.combination.optimal=최적 조합(순위 1)이 선정되었습니다. 실질 환급 효과: {0}원
```

#### errors_ko.properties (오류 메시지)

```properties
# === 입력 검증 (400) ===
error.invalid.json=잘못된 JSON 형식입니다. 요청 데이터를 확인해 주십시오.
error.validation.failed=입력 데이터 검증에 실패하였습니다.
error.missing.required.category=필수 카테고리 '{0}'이(가) 누락되었습니다.
error.invalid.tax.type=세목 구분 '{0}'은(는) 유효하지 않습니다. CORP 또는 INC를 입력해 주십시오.
error.invalid.req.id.format=요청번호 형식이 올바르지 않습니다.
error.payload.too.large=요청 데이터가 허용 크기({0}MB)를 초과하였습니다.

# === 비즈니스 (422) ===
error.hard.fail=경정청구 불가 사유가 발견되었습니다: {0}
error.hard.fail.deadline=경정청구 기한(법정신고기한 + 5년)이 경과하였습니다.
error.hard.fail.settlement=결산확정 항목(감가상각비/퇴직급여/대손충당금)의 추가 계상은 경정청구 대상이 아닙니다.
error.hard.fail.estimate=추계신고자는 조세특례제한법상 감면/공제가 배제됩니다.
error.calculation.failed=환급액 계산 중 오류가 발생하였습니다: {0}
error.convergence.failed=과세표준-최저한세 순환참조가 수렴하지 못하였습니다.

# === 리소스 (404) ===
error.request.not.found=요청번호 '{0}'에 해당하는 경정청구 요청을 찾을 수 없습니다.
error.report.not.found=요청번호 '{0}'의 보고서가 아직 생성되지 않았습니다.

# === 상태/중복 (409) ===
error.invalid.status=현재 상태({0})에서는 해당 작업을 수행할 수 없습니다.
error.duplicate.request=동일한 요청이 이미 처리되었습니다. 기존 요청번호: {0}

# === 시스템 (500/504) ===
error.timeout=처리 시간이 초과되었습니다. 잠시 후 다시 시도해 주십시오.
error.internal=시스템 내부 오류가 발생하였습니다. 관리자에게 문의해 주십시오.
error.database=데이터베이스 처리 중 오류가 발생하였습니다.
```

#### validation_ko.properties (검증 메시지)

```properties
# === 107개 검증 규칙 메시지 (주요 항목) ===
validation.V01=경정청구 기한({0})이 경과하였습니다. 법정신고기한 + 5년 이내만 가능합니다.
validation.V02=요청번호(req_id) 형식이 올바르지 않습니다: {0}
validation.V03=세목 구분(tax_type)은 CORP 또는 INC만 허용됩니다.
validation.V04=필수 필드 '{0}'이(가) 누락되었습니다.
validation.C01=결산확정 항목은 경정청구 대상이 아닙니다.
validation.C02=산출세액은 0 이상이어야 합니다.
validation.C03=공제액({0}원)이 산출세액({1}원)을 초과할 수 없습니다.
validation.B01=상호배제 규칙 위반: {0}과(와) {1}은(는) 동시 적용이 불가합니다.
validation.L01=최저한세({0}원) 한도를 초과하였습니다. 초과분은 이월 처리됩니다.
validation.L02=적용순서 위반: 감면 → 이월불가 공제 → 이월가능 공제 순서를 준수해야 합니다.
validation.W01=순환참조가 5회 내 수렴하지 못했습니다. 현재까지 최선 결과를 적용합니다.
validation.W02=동일 업종 5년 내 폐업 이력이 있습니다. 재창업 요건을 재확인해 주십시오.
```

---

## 11. 환급액 계산 로직 상세 설계

### 11.1 전체 계산 파이프라인

```
[TX-1: 입력 트랜잭션] ────────────────────────────────────────
  M1-01: req_id 발급 → INP_REQUEST
  M1-02: JSON 원본 보관 → INP_RAW_DATASET (INSERT ONLY)
  M1-03: 요약 생성 → SMR_BASIC_INFO / SMR_EMPLOYEE /
                      SMR_DEDUCTION_ITEM / SMR_FINANCIAL
  M1-04~12: 카테고리별 상세 검증 (CORP)
  M1-13: 감면 변경 경정청구 판별 (amendment_type = 'CHANGE' 시)
         → 기존 감면 취소 처리 + 신규 감면 적용 분기
         예: §7→§6 전환, 증가분→당기분 방식 변경, 공제항목 추가/제거
         → INP_S_EXISTING_DEDUCTION에서 기존 공제 이력 로드 후 차액 산출
  P1-01~08: 카테고리별 상세 검증 (INC)
  COMMIT TX-1

[TX-2: 계산 트랜잭션] ────────────────────────────────────────
  M3-PREP: 데이터 준비 → SMR_PREP
  M3-00~06: 사전 점검 → SMR_ELIGIBILITY / SMR_VALIDATION_LOG
  P3-01~04: 개인 전용 점검 (INC)

  M4-01~06: 6대 핵심 공제/감면 → OUT_CREDIT_DETAIL
    * CreditCalculator Strategy Pattern (25개 서브모듈)
    * 법인 전용: M4-07~M4-42
    * 개인 전용: P4-01~P4-12

  M5-01: 상호배제 검증 → OUT_EXCLUSION_VERIFY
         ┌──────────────────────────────────────────────────────────────┐
         │ 상호배제 10규칙 (조특법 §127④ 기준)                          │
         ├──────┬──────────────┬──────────────┬────┬─────────────────────┤
         │ Rule │ 조합 A        │ 조합 B        │중복│ 조건/근거            │
         ├──────┼──────────────┼──────────────┼────┼─────────────────────┤
         │  1   │ SS6 (창업)    │ SS7 (중소특별) │ X  │ 택1 선택 (§127④)    │
         │  2   │ SS6 (창업)    │ SS29의8 (고용) │조건│ 2024이전:O/2025이후:X│
         │  3   │ SS7 (중소특별) │ SS29의8 (고용) │ O  │ §127④ 괄호 규정     │
         │  4   │ SS6 (창업)    │ SS24 (투자)   │ X  │ 중복 불가 (§127④)   │
         │  5   │ SS7 (중소특별) │ SS24 (투자)   │ X  │ 중복 불가 (§127④)   │
         │  6   │ SS10 (R&D)   │ 모든 항목      │ O  │ §127 대상 外         │
         │      │              │              │    │ (동일자산 SS24 제외)  │
         │  7   │ SS7 (중소특별) │ SS30의4 (사보) │ O  │ §127④ 괄호 규정     │
         │  8   │ SS6 (창업)    │ SS30의4 (사보) │ X  │ 중복 불가 (국세상담)  │
         │  9   │ SS6§7항(추가) │ SS29의8 (고용) │ X  │ §127④ 단서          │
         │ 10   │ SS24 (투자)   │ SS10 (R&D)   │조건│ 동일자산만 중복 불가  │
         └──────┴──────────────┴──────────────┴────┴─────────────────────┘
         ※ 핵심 조문 (조특법 §127④):
           "제6조,제7조에 따라 감면되는 경우와 제24조,제30조의4
            [제7조와 동시에 적용되는 경우는 제외]에 따라 공제되는
            경우를 동시에 적용받을 수 있는 경우에는 그 중 하나만을
            선택하여 적용받을 수 있다."
         ※ Rule 4,5: 참고1번(v2.1)에서 "병행 가능"으로 기재되어
           있었으나 프롬프트 법조문 확인 결과 "중복 불가"로 정정.
  M5-02: 최적 조합 탐색 (B&B / Greedy) → OUT_COMBINATION
  M5-02a: [INC] 소득공제 ↔ 세율구간 최적화 (소득공제 적용 후 과세표준이
           세율구간 경계를 넘을 경우 추가 절세효과 산출 → 최적 소득공제 조합 탐색)
  M5-03: 최저한세 적용
  M5-04: 농특세 적용
  M5-05: 적용순서 처리
  MX-01: 순환참조 해결 (최대 5회)

  M6-01: 환급액 비교표 → OUT_REFUND
  M6-02: 환급가산금 → OUT_REFUND
         ※ 법인: "환급가산금은 법인세법상 익금불산입 대상입니다" 안내 문구 포함
         ※ 개인: "환급가산금은 종합소득세 과세대상이 아닙니다" 안내 문구 포함
  M6-03: 지방소득세 안내
  M6-04: 보고서 3종
  M6-04a: 사후관리 리스크 안내 섹션 (감면별 사후관리기간, 요건, 위반시 추징액 예상)
          ┌──────────────────────────────────────────────────────┐
          │ 감면 항목    │ 사후관리 기간 │ 핵심 요건     │ 추징 시 예상액 │
          ├──────────────┼──────────────┼──────────────┼────────────────┤
          │ §6 창업감면   │ 5년          │ 업종유지/폐업금지│ F-COM-01 적용 │
          │ §24 투자공제  │ 2~5년        │ 자산처분금지   │ 공제세액 환수  │
          │ §29의8 고용   │ 2년          │ 상시근로자유지  │ F-COM-01 적용 │
          │ §10 R&D      │ 2년          │ R&D비 유지    │ 공제세액 환수  │
          └──────────────┴──────────────┴──────────────┴────────────────┘
  M6-05: JSON 8섹션 직렬화 → OUT_REPORT_JSON
  COMMIT TX-2
```

### 11.2 CreditCalculator Strategy Pattern

```java
/**
 * 공제/감면 계산 Strategy 인터페이스.
 *
 * <p>25개 서브모듈이 이 인터페이스를 구현한다.
 * 각 Calculator는 독립적으로 테스트 가능하며,
 * CreditCalculatorRegistry를 통해 자동 등록된다.</p>
 *
 * @see CreditCalculatorRegistry 레지스트리
 * @see CalculationContext 계산 컨텍스트
 */
public interface CreditCalculator {
    /**
     * 적용 조항 코드를 반환한다.
     * @return 조항 코드 (예: "SS24", "SS29_8")
     */
    String getProvision();

    /**
     * 지원하는 세목 타입을 반환한다.
     * @return 세목 집합 (CORP, INC, 또는 양쪽)
     */
    Set<TaxType> getSupportedTaxTypes();

    /**
     * 공제/감면 금액을 계산한다.
     *
     * @param ctx 계산 컨텍스트 (요약 데이터, 기준정보 포함)
     * @return 계산 결과 (총액, 절사액, 농특세, 최저한세 대상 등)
     */
    CreditResult calculate(CalculationContext ctx);
}
```

### 11.2.1 EmploymentCreditCalc 상세 — 청년 판단 및 경과규정

```
[청년 판단 로직 — YouthDetermination]
  STEP 1: 기본연령 = 판단연도 - 출생연도
  STEP 2: 병역보정연령 = 기본연령 - TRUNCATE(병역이행개월수 / 12, 0)
  STEP 3: 병역한도 = MIN(병역이행개월수, 72)  // 최대 6년
  STEP 4: 청년여부 = (병역보정연령 ≤ 34) AND (기본연령 ≤ 40)

  ※ 2024 세법개정: 병역가산연령 상한 40세 신설 (2024.1.1 이후 과세연도)
  ※ 2023 이전: 병역가산 후 연령 34세 이하만 판단 (40세 절대한도 없음)

[고용증대세액공제 경과규정 — §29의7 (2022 폐지)]
  조건: tax_year ≤ 2021 발생분 + 이월공제 잔액 존재
  분기: provision_code == "SS29_7"인 INP_S_EXISTING_DEDUCTION 행 식별
  처리: 이월공제 5년한도 내 잔액 → 당기 세액에서 차감
  ※ EmploymentCreditCalc.calculate() 내 §29의7 경과규정 분기 포함
  ※ 신규 발생은 §29의8(통합고용)만 허용, §29의7은 이월분만 처리
```

### 11.2.2 RdCreditCalc 상세 — 최적 방식 선택 로직

```
[R&D 세액공제 방식 선택 — selectOptimalMethod()]
  STEP 1: 증가분 방식 산출
    증가분공제 = MAX(0, 당기R&D비 - 직전연도R&D비) × 증가분공제율
    (증가분공제율: 중소50%, 중견40%, 대25% — 신성장·원천/국가전략 별도)

  STEP 2: 당기분 방식 산출
    당기분공제 = 당기R&D비 × 당기분공제율
    (당기분공제율: 중소25%, 중견8~15%, 대0~2%)

  STEP 3: 유리 선택
    최종공제액 = MAX(증가분공제, 당기분공제)
    선택방식 = (증가분공제 ≥ 당기분공제) ? "INCREMENTAL" : "CURRENT"

  ※ R&D 유형별 분기: 일반 / 신성장·원천 / 국가전략
  ※ 국가전략기술: 증가분 30~40%, 당기분 20~30% (별도 우대 공제율)
  ※ 선택 결과를 OUT_CREDIT_DETAIL.method_type에 기록
```

### 11.2.3 CorpRestructuringCalc 상세 — 기업구조조정 과세이연 (M4-10)

```
[기업구조조정 과세이연 — §44~47]
  대상: 합병(§44), 분할(§46), 현물출자(§47), 포괄양수도(§44①3)

  STEP 1: 적격요건 검증
    - 사업계속요건 (피합병법인 근로자 80% 이상 승계)
    - 지분연속성요건 (합병대가 중 주식 80% 이상)
    - 사업지속요건 (합병법인이 2년 이상 사업 영위)

  STEP 2: 과세이연 금액 산출
    양도차익 = 시가 - 장부가
    이연금액 = 적격요건 충족 시 양도차익 전액 이연
    비적격 시 = 양도차익 전액 익금산입

  STEP 3: 사후관리 기간 검증
    - 사후관리기간: 합병등기일로부터 5년
    - 위반 시 추징: 이연세액 + 이자상당가산액

  입력: INP_CORP_RESTRUCTURING (합병유형, 적격요건항목, 양도차익, 장부가)
  출력: OUT_CREDIT_DETAIL (provision='RESTRUCTURING', 이연금액, 사후관리만료일)
```

### 11.2.4 ConsolidatedTaxCalc 상세 — 연결납세 (M4-11)

```
[연결납세 — §76의8]
  대상: 연결모법인 + 연결자법인 (100% 출자)

  STEP 1: 연결법인 식별
    - INP_CORP_BASIC.consolidated_group_id로 그룹 식별
    - consolidated_yn = 'Y'인 법인만 대상

  STEP 2: 연결과세표준 산출
    연결소득 = SUM(각 연결법인 개별소득)
    연결조정 = 내부거래 제거 + 연결결손금 통산
    연결과세표준 = 연결소득 + 연결조정 - 연결이월결손금

  STEP 3: 환급세액 배분
    개별법인 환급 = 연결환급세액 × (개별법인세액 / 연결총세액)

  DB 추가: INP_CORP_BASIC에 consolidated_yn(CHAR 1), consolidated_group_id(VARCHAR 20) 컬럼
```

### 11.3 전체 계산 공식 목록 (57개)

> 참고1번 6절 기반. 법인 F01~F41(21개) + 개인 F-INC-01~12(12개) + 공통 F-COM-01(1개) = 총 57개 공식 (기존 53개에서 F11, F-COM-01, F-INC-07~12 추가). F번호는 비연속(F01~F04, F10~F13, F14~F19, F30~F34, F40~F41)이므로 41번까지이나 실제 21개.

#### 11.3.1 법인세 과세표준 및 산출세액 (F01~F04)

| 공식 ID | 공식명 | 산출 공식 | 절사 | 근거 |
|---------|--------|----------|------|------|
| F01 | 과세표준 | 각사업연도소득 - 이월결손금 - 비과세소득 | 10원 | 법인세법 §13 |
| F02 | 이월결손금 공제한도 | MIN(잔액, 소득 × 한도율) [중소 100%, 기타 80%] | — | 법인세법 §13 |
| F03 | 산출세액 | 과세표준 × 초과누진세율 (9~24% 4단계, 연도별 분기) | 10원 | 법인세법 §55 |
| F04 | 최저한세 | 과세표준 × 최저한세율 (아래 11.3.6 구간별 상세) | 10원 | 조특법 §132 |

#### 11.3.2 공제/감면 세액 (F10~F19)

| 공식 ID | 공식명 | 산출 공식 | 절사 | 근거 |
|---------|--------|----------|------|------|
| F10 | 통합고용 기본 | (청년등증가 × 단가) + (일반증가 × 단가) | 10원 | §29의8 |
| F11 | 통합고용 추가 | SUM(경력단절×1,300만/900만 + 정규직전환×1,300만/900만 + 육아복귀×1,300만/900만) | 10원 | §29의8 |
| F12 | 통합투자 기본 | 투자금액 × 기본공제율 (중소10%/중견3%/대1%) | 10원 | §24 |
| F13 | 통합투자 추가 | MAX(0, 당해투자-직전3년평균) × 추가율 (2025 이후 10% 상향) | 10원 | §24 |
| F14 | 창업감면 | 산출세액 × (감면소득/총소득) × 감면율 (50~100%) | 10원 | §6 |
| F15 | 중소특별감면 | 산출세액 × (감면소득/총소득) × 감면율 (5~30%) | 10원 | §7 |
| F16 | 중소특별 감면한도 | MIN(감면액, MAX(0, 1억 - 감소인원×500만)) | 10원 | §7 |
| F17 | R&D 증가분 | (해당R&D비 - 직전R&D비) × 공제율 | 10원 | §10 |
| F18 | R&D 당기분 | 해당R&D비 × 공제율 (중소25%/중견8~15%/대0~2%) | 10원 | §10 |
| F19 | 사회보험료 공제 | SUM(사업주부담분 × 공제율, 근로자별) [청년100%/일반50%] | 10원 | §30의4 [Phase 2] |

#### 11.3.3 최종 환급액 (F30~F34)

| 공식 ID | 공식명 | 산출 공식 | 절사 | 근거 |
|---------|--------|----------|------|------|
| F30 | 법인세 환급액 | 기존납부세액 - 경정후결정세액 | — | — |
| F31 | 환급가산금(본세) | 환급액 × 이율 × 일수/365 | 1원 | 국기법 §52 |
| F32 | 환급가산금(중간예납) | 중간예납과납 × 이율 × 일수/365 | 1원 | 국기법 §52 |
| F33 | 지방소득세 환급 | 법인세환급액 × 10% | 10원 | 지방세법 §103의23 |
| F34 | 총수령예상액 | F30 + F31 + F32 + F33 | — | — |

#### 11.3.4 농특세 (F40~F41)

| 공식 ID | 공식명 | 산출 공식 | 절사 | 근거 |
|---------|--------|----------|------|------|
| F40 | 농특세 산출 | 과세대상공제액 × 20% | 10원 | 농어촌특별세법 §5 |
| F41 | 농특세 변동 | 경정후 농특세 - 기존 농특세 (조합 변경 시 연쇄 재계산) | 10원 | — |

> **농특세 비과세/과세 구분**: SS6(창업), SS7(중소특별), SS10(R&D) = **비과세** / SS24(투자), SS29의8(고용), SS30의4(사보) = **과세 20%**

#### 11.3.5 종합소득세 계산 공식 (F-INC-01~F-INC-12)

| 공식 ID | 공식명 | 산출 공식 | 절사 | 근거 |
|---------|--------|----------|------|------|
| F-INC-01 | 종합소득금액 | 사업소득 + 근로소득 + 금융소득(2천만초과시) + 연금 + 기타 | — | 소득세법 §14 |
| F-INC-02 | 과세표준 | 종합소득금액 - 소득공제합계 | 10원 | 소득세법 §14 |
| F-INC-03 | 산출세액 | 과세표준 × 초과누진세율 (6~45% 8단계, 연도별 분기) | 10원 | 소득세법 §55 |
| F-INC-04 | 감면소득배분 | 산출세액 × (감면대상소득/종합소득금액) × 감면율 | 10원 | 시행령 §117 |
| F-INC-05 | 공동사업자 배분 | 전체소득 × 손익분배비율 | — | 소득세법 §43 |
| F-INC-06 | 개인 최저한세 | 3천만이하: 산출세액×35% / 초과: 산출세액×45% | 10원 | 조특법 §132② |
| F-INC-07 | 공제가능한도 | 산출세액 - 최저한세액 | — | — |
| F-INC-08 | 개인 환급액 | 실제납부세액 - 경정후 결정세액 | — | — |
| F-INC-09 | 개인 환급가산금 | 환급액 × 이율 × 일수/365 (기산일: 5.31 또는 성실 6.30) | 1원 | 국기법 §52 |
| F-INC-10 | 개인 총수령예상액 | F-INC-08 + F-INC-09 + 지방소득세환급 | — | — |
| F-INC-11 | 소급공제 환급세액 | 직전연도 산출세액 × (소급결손금/직전연도 과세표준) | 10원 | 소득세법 §85의2 |
| F-INC-12 | 전자신고 세액공제 | 20,000원 (정액, 전자신고 이력 확인 시) | — | 조특법 §104의8 |

#### 11.3.6 사후관리 추징액 공식 (F-COM-01)

> 고용세액공제(SS29의8) 사후관리 위반 시 추징액 산출. 법인/개인 동일 적용.

```
F-COM-01: 고용세액공제 추징액 (이자상당액 포함)

STEP 1: 본세 추징액 = 감소인원 × 인당공제액 (청년/일반 구분)  [10원 절사]
STEP 2: 경과일수 = 추징사유발생일 - (공제과세연도종료일 + 1일)
STEP 3: 이자상당가산액 = TRUNCATE(본세추징액 × 경과일수/365 × 이율, -1)  [10원 절사]
STEP 4: 총추징액 = 본세추징액 + 이자상당가산액

※ 이자율: 국세기본법 시행규칙 §19의3 (1일 0.022%, 연 약 8.03%)
※ 퇴사예외사유(resign_exception_code) 해당 시 추징 제외
```

#### 11.3.7 법인 최저한세율 구간별 상세 (조특법 §132)

| 기업 규모 | 과세표준 구간 | 최저한세율 |
|----------|:----------:|:--------:|
| **중소기업** | 전 구간 | **7%** |
| **중견기업** | ~100억 | **10%** |
| **중견기업** | 100억~1,000억 | **12%** |
| **대기업** | ~100억 | **10%** |
| **대기업** | 100억~1,000억 | **12%** |
| **대기업** | 1,000억 초과 | **17%** |

#### 11.3.8 R&D 최저한세 배제율 3단계 (REF_C_RD_MIN_TAX_EXEMPT)

> M5-03에서 R&D 공제분을 별도 분리하여 배제율 적용

| R&D 유형 | 대상 기업 | 배제율 | 의미 |
|---------|----------|:-----:|------|
| **국가전략기술** | 전체(중소/중견/대) | **100%** | 전액 최저한세 적용 제외 |
| **신성장·원천기술** | 중소기업 | **100%** | 전액 최저한세 적용 제외 |
| **신성장·원천기술** | 중견·대기업 | **0%** | 전액 최저한세 적용 |
| **일반 R&D** | 중소기업 | **50%** | 공제액의 50%만 최저한세 배제 |
| **일반 R&D** | 중견·대기업 | **0%** | 전액 최저한세 적용 |

---

## 12. 데이터베이스 테이블 설계

> 상세 스키마는 `docs/01-plan/schema.md` 참조 (63개 테이블, 5계층)

### 12.1 핵심 설계 원칙

| 원칙 | 설명 |
|------|------|
| **req_id 관통** | 기준정보(REF_*)를 제외한 모든 테이블에 req_id FK |
| **3계층 분리** | INP_(입력 보관) / SMR_(중간 요약) / OUT_(출력 보관) 물리 분리 |
| **JSON-In / JSON-Out** | INP_RAW_DATASET은 JSONB 원본 보관, OUT_REPORT_JSON은 8섹션 JSON 직렬화 |
| **원본 불변성** | INP_RAW_DATASET은 INSERT ONLY (DB 트리거 차단) |
| **요약 재생성** | SMR_*는 INP_RAW_DATASET에서 언제든 재파싱 가능 |
| **기준정보 독립** | REF_*는 req_id 없이 연도별 독립 관리 |

### 12.2 테이블 요약 (63개)

| 계층 | 접두어 | 테이블 수 | 역할 |
|------|--------|:--------:|------|
| **Layer 1: 입력** | INP_ | 20 | JSON 데이터셋 원본 보관 + 파싱된 상세 |
| **Layer 2: 요약** | SMR_ | 7 | 기준정보 활용 계산용 중간 요약 |
| **Layer 3: 출력** | OUT_ | 9 | 산출 결과, JSON 직렬화 대상 |
| **Layer 4: 기준정보** | REF_ | 26 | 세율/공제율/한도 등 독립 기준 (req_id 무관) |
| **Layer 5: 감사** | AUD_ | 1 | 계산 이력 APPEND ONLY |

### 12.3 INSERT ONLY 트리거 (DDL)

```sql
-- INP_RAW_DATASET: UPDATE/DELETE 차단 트리거
CREATE OR REPLACE FUNCTION prevent_raw_data_modification()
RETURNS TRIGGER AS $$
BEGIN
    RAISE EXCEPTION 'INP_RAW_DATASET은 INSERT ONLY입니다. 수정/삭제할 수 없습니다. (req_id: %, category: %)',
        OLD.req_id, OLD.category;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_raw_data_no_update
    BEFORE UPDATE OR DELETE ON inp_raw_dataset
    FOR EACH ROW EXECUTE FUNCTION prevent_raw_data_modification();

-- AUD_CALCULATION_LOG: UPDATE/DELETE 차단 트리거
CREATE TRIGGER trg_audit_log_no_update
    BEFORE UPDATE OR DELETE ON aud_calculation_log
    FOR EACH ROW EXECUTE FUNCTION prevent_raw_data_modification();
```

### 12.4 REF_* 기준정보 갱신 프로세스

> 참고1번 10.3절 기준, 9개 핵심 기준정보 테이블의 갱신 절차를 정의한다.

#### 갱신 대상 및 주기

| 테이블 | 갱신 주기 | 담당 | 트리거 | 비고 |
|--------|:--------:|------|--------|------|
| REF_S_TAX_RATE_BRACKET | 연 1회 | ADMIN | 세법 개정 공포 | 법인세/소득세율 구간 |
| REF_S_MIN_TAX_RATE | 연 1회 | ADMIN | 세법 개정 공포 | 최저한세율 구간 |
| REF_S_MUTUAL_EXCLUSION | 연 1회 | ADMIN | 세법 개정 공포 | 상호배제 규칙 |
| REF_S_INDUSTRY_ELIGIBILITY | 연 1회 | ADMIN | 업종 코드 변경 | 업종별 적격성 |
| REF_C_INVESTMENT_CREDIT_RATE | 연 1회 | ADMIN | 세법 개정 공포 | 투자공제율 |
| REF_S_EMPLOYMENT_CREDIT | 연 1회 | ADMIN | 세법 개정 공포 | 고용공제 단가 |
| REF_C_STARTUP_EXEMPTION | 연 1회 | ADMIN | 세법 개정 공포 | 창업감면율 |
| REF_C_RD_MIN_TAX_EXEMPT | 연 1회 | ADMIN | 세법 개정 공포 | R&D 최저한세 배제율 |
| REF_S_REFUND_INTEREST_RATE | 수시 | ADMIN | 국세기본법 시행규칙 | 환급가산금 이율 |

#### 갱신 절차

```
1. [준비] ADMIN이 관보/국세법령정보시스템에서 개정 내용 확인
2. [입력] ReferenceDataService.createNewVersion(tableName, taxYear, data)
   → 기존 데이터는 유지, 새 tax_year 행 INSERT (히스토리 보존)
3. [검증] 이전 연도 데이터와 비교 미리보기
   → diff(previousYear, newYear) → 변경 필드/값 리스트 출력
4. [승인] ADMIN 확인 후 활성화 (active_yn = 'Y')
5. [감사] AUD_CALCULATION_LOG에 갱신 이력 기록
   → 갱신자, 갱신일시, 변경 전후 값
```

#### 갱신 검증 체크리스트

| 검증 항목 | 방법 |
|-----------|------|
| 세율 구간 연속성 | 과세표준 구간이 겹치거나 누락 없는지 확인 |
| 공제율 범위 검증 | 0% ≤ rate ≤ 100% |
| 연도 중복 방지 | (provision_code, tax_year) 유니크 제약 |
| 이전 연도 대비 변동폭 | 50% 이상 변동 시 경고 |
| 상호배제 규칙 무결성 | 양방향 대칭 (A↔B) 검증 |

---

## 13. REST API 설계

### 13.1 엔드포인트 목록

| No | Method | Path | 설명 | TX | 권한 |
|----|--------|------|------|:--:|------|
| API-01 | POST | `/api/v1/requests` | 요청 접수 + JSON 수신 | TX-1 | ACCOUNTANT+ |
| API-02 | POST | `/api/v1/requests/{req_id}/analyze` | 점검 실행 | TX-2 | ACCOUNTANT+ |
| API-03 | GET | `/api/v1/requests/{req_id}/status` | 상태 조회 | 읽기 | VIEWER+ |
| API-04 | GET | `/api/v1/requests/{req_id}/report` | 전체 보고서 JSON | 읽기 | VIEWER+ |
| API-05 | GET | `/api/v1/requests/{req_id}/report/sections/{A-H}` | 섹션별 JSON | 읽기 | VIEWER+ |
| API-06 | GET | `/api/v1/requests/{req_id}/raw-data` | 원시 입력 조회 | 읽기 | ACCOUNTANT+ |
| API-07 | GET | `/api/v1/reference/exclusion-matrix` | 상호배제 기준정보 | 읽기 | VIEWER+ |
| API-08 | GET | `/api/v1/requests/{req_id}/summary` | 경량 요약 조회 | 읽기 | VIEWER+ |
| API-09 | GET | `/api/v1/requests/{req_id}/credits` | 공제/감면 상세 | 읽기 | VIEWER+ |
| API-10 | GET | `/api/v1/requests/{req_id}/combinations` | 조합 비교 | 읽기 | VIEWER+ |
| API-11 | GET | `/api/v1/requests/{req_id}/audit-log` | 감사 로그 | 읽기 | ADMIN |

### 13.2 API-01 상세 (요청 접수)

**요청 헤더**:
```
POST /api/v1/requests
Content-Type: application/json
Authorization: Bearer {JWT}
X-Idempotency-Key: {UUID}
```

**요청 Body**: `AmendmentClaimRequest` DTO (섹션 6.3 참조)

**성공 응답 (201 Created)**:
```json
{
  "reqId": "C-1234567890-20260218-001",
  "status": "received",
  "datasetsReceived": 3,
  "createdAt": "2026-02-18T14:00:00+09:00"
}
```

**오류 응답**: `ErrorResponse` DTO (섹션 6.3 참조)

---

## 14. 트랜잭션 및 동시성 설계

### 14.1 트랜잭션 경계

| TX | 범위 | 격리수준 | 타임아웃 | 성공 | 실패 |
|:--:|------|---------|:-------:|------|------|
| TX-1 | M1-01 ~ M1-12 / P1-01~08 | READ COMMITTED | 60초 | COMMIT → parsed | ROLLBACK → error |
| TX-2 | M3-PREP ~ M6-05 | READ COMMITTED | 300초 | COMMIT → completed | ROLLBACK → error |

### 14.2 동시성 제어

| 대상 | 방식 | 설명 |
|------|------|------|
| req_id 발급 | SELECT FOR UPDATE | 동일 (applicant_id, request_date) 행 수준 락 |
| seq_no 충돌 | UNIQUE INDEX | 멱등성 키 + 날짜 유니크 |
| 데드락 | 지수 백오프 재시도 | 최대 3회, 100ms × 2^n |
| Idempotency | X-Idempotency-Key | 24시간 TTL, 동일 키 → 기존 결과 반환 |

### 14.3 성능 제한

| 항목 | 제한값 |
|------|-------|
| 카테고리별 JSON 크기 | 10 MB |
| 최대 카테고리 수/요청 | 40개 |
| 전체 페이로드 | 50 MB |
| employee_detail 최대 | 10,000건 |
| M5 조합 탐색 타임아웃 | 120초 |
| 77개 항목 전체 산출 | 30초 이내 |

---

## 15. 보안 설계

### 15.1 보안 요건

| 항목 | 방식 |
|------|------|
| API 인증 | JWT (8시간 유효, RBAC 4단계) |
| 개인정보 암호화 | AES-256 (주민번호, 사업자번호) |
| 통신 보안 | HTTPS TLS 1.2+ |
| 데이터 보관 | 90일 자동 삭제 |
| 로그 마스킹 | 사업자번호/대표자 식별정보 마스킹 |
| INSERT ONLY | INP_RAW_DATASET DB 트리거 차단 |
| 감사 추적 | AUD_CALCULATION_LOG APPEND ONLY |
| 멱등성 | X-Idempotency-Key 중복 방지 |

---

## 16. 소스코드 주석 컨벤션

### 16.1 클래스 주석 (Javadoc)

모든 클래스에 다음을 포함한다:

```java
/**
 * [클래스 한글명] - [영문명].
 *
 * <p>[클래스의 역할과 책임을 상세히 기술한다.]</p>
 *
 * <p>[관련 비즈니스 규칙, 설계 결정 사항, 주의사항 등을 기술한다.]</p>
 *
 * <p>사용 예:
 * <pre>
 *   [코드 사용 예시]
 * </pre></p>
 *
 * @author [작성자]
 * @version [버전]
 * @since [작성일]
 * @see [관련 클래스/인터페이스 참조]
 */
```

### 16.2 메서드 주석 (Javadoc)

모든 public 메서드에 다음을 포함한다:

```java
/**
 * [메서드의 역할을 상세히 기술한다.]
 *
 * <p>[비즈니스 규칙, 계산 공식, 참조 법령 등을 기술한다.]</p>
 *
 * @param paramName [입력값의 의미, 타입, 제약조건, 예시]
 * @return [출력값의 의미, 타입, 가능한 값 범위]
 * @throws ExceptionType [예외 발생 조건]
 * @see [관련 메서드/클래스]
 */
```

### 16.3 인라인 주석

```java
// === 단계 1: 이월결손금 잔액 확인 (FIFO 순서) ===
// 2020.1.1 이후 발생 → 15년 이월, 이전 → 10년 이월 (법인세법 §13)
long remainingLoss = calculateRemainingLoss(losses, taxYear);

// 중소기업: 소득의 100% 공제 가능 / 비중소기업: 80% 한도
double limitRate = isSme ? 1.0 : 0.8;

// 10원 미만 절사 (국세기본법 §86) — 반올림 절대 금지
long deductible = TruncationUtil.truncateAmount(
    (long)(totalIncome * limitRate));
```

---

## 17. 테스트 계획

### 17.1 단위 테스트 (57개 공식)

모든 계산 공식(F01~F41, F-INC-01~12, F-COM-01)은 JUnit 5 + AssertJ로 100% 커버리지 달성 필수.

| 테스트 클래스 | 대상 공식 | 시나리오 수 | 비고 |
|-------------|---------|:--------:|------|
| InvestmentCreditCalcTest | F12, F13 | 12+ | 기본+추가, 2025 추가율 상향, 과밀억제 |
| EmploymentCreditCalcTest | F10, F11 | 15+ | 청년/일반, 경력단절/정규직/육아, 경과규정 |
| StartupExemptionCalcTest | F14 | 8+ | 감면율 분기(청년/비청년/벤처/인구감소) |
| SmeSpecialExemptionCalcTest | F15, F16 | 8+ | 감면율+한도+근로자감소 조정 |
| RdCreditCalcTest | F17, F18 | 10+ | 증가분/당기분 비교, 5개년 연속성 |
| MinimumTaxServiceTest | F04, F-INC-06 | 10+ | 법인 구간별, 개인 35%/45%, R&D 배제 |
| TruncationUtilTest | 절사 전체 | 8+ | 10원/1원/비율/근로자 수 절사 |
| MutualExclusionServiceTest | 10규칙 | 20+ | 연도별 분기(2024/2025), 모든 조합 |

### 17.2 E2E 시나리오 (10개)

#### 법인 시나리오 5건

| No | 시나리오 | 핵심 검증 포인트 |
|:--:|---------|:-------------|
| C-01 | 중소기업 제조업, 투자+고용 병행 | SS24+SS29의8 중복 허용, 농특세 20%, 최저한세 7% |
| C-02 | 중소기업 창업감면 vs 중소특별, 택1 | SS6/SS7 상호배제, B&B 최적 선택 |
| C-03 | 중견기업 R&D 집중, 투자 소액 | R&D 최저한세 배제율, SS10+SS24 동일자산 배제 |
| C-04 | 대기업 다항목 복합 | 최저한세 17%, Greedy 폴백, 대규모 조합 |
| C-05 | 법인 2024+2025 경과규정 | 2024이전 SS6+SS29의8=O, 2025이후 SS6+SS29의8=X |

#### 개인 시나리오 5건

| No | 시나리오 | 핵심 검증 포인트 |
|:--:|---------|:-------------|
| I-01 | 개인 단일사업장, 중소기업, 고용+투자 | 대표자 제외, 산출세액 기준 최저한세 35% |
| I-02 | 다사업장(3개), 감면소득 배분 | F-INC-04 배분, 사업장별 소재지 개별 판단 |
| I-03 | 공동사업자, 소득 배분 | F-INC-05 손익분배비율, P1-08 검증 |
| I-04 | 성실사업자, 의료비/교육비/확인비용 | P4-04 최저한세 비대상, P4-05 최저한세 대상 구분 |
| I-05 | 개인 결손금 소급공제 | F-INC-11 NPV 비교, 이월 vs 소급 선택 |

### 17.3 부하 테스트 (성능 검증)

| 테스트 항목 | 도구 | 기준 | 방법 |
|-----------|------|------|------|
| 77개 항목 전체 산출 | JMeter | **30초 이내** | 중소기업 표준 데이터셋 기준 |
| 동시 100 요청 처리 | JMeter | 평균 응답 60초 이내 | 100 가상유저, 60초 Ramp-up |
| DB 커넥션 풀 안정성 | JMeter | 에러율 0% | 300초 지속 부하 |
| B&B 탐색 타임아웃 | 단위 테스트 | 120초 이내 | 15개 항목 최악 케이스 |

### 17.4 SonarQube CI/CD 통합

```
Pipeline: Git Push → SonarQube Scan → Quality Gate → Build → Deploy
Quality Gate 기준:
  - 신규 코드 커버리지 ≥ 80%
  - 기술부채 등급 ≥ A
  - 치명적 이슈 0건
  - 주요 이슈 0건
```

---

## 18. 가용성 및 모니터링 설계

### 18.1 가용성 SLA 목표

| 항목 | 기준 |
|------|------|
| SLA 목표 | **99.5%** (연간 최대 43.8시간 다운타임) |
| TX ACID | TX-1, TX-2 모두 ACID 완전 보장 |
| 장애 복구 시간(RTO) | 30분 이내 |
| 데이터 복구 시점(RPO) | 최근 1시간 이내 |

### 18.2 모니터링 및 알림

| 항목 | 도구 | 임계치 | 알림 |
|------|------|--------|------|
| API 응답 시간 | Spring Actuator + Micrometer | p95 > 30초 | WARNING |
| 에러율 | Logback + 알림 연동 | 5분 내 3건 이상 | CRITICAL |
| DB 커넥션 풀 | HikariCP 메트릭 | 사용률 > 80% | WARNING |
| 디스크/메모리 | OS 메트릭 | 사용률 > 85% | WARNING |
| TX-2 타임아웃 | 애플리케이션 로그 | 발생 시 | CRITICAL |

### 18.3 장애 복구 전략

| 전략 | 적용 대상 |
|------|----------|
| DB 자동 백업 (일 1회) | PostgreSQL pg_dump |
| 재시도 (지수 백오프) | 데드락, 네트워크 일시 장애 |
| Circuit Breaker | 외부 연동 시 (Phase 2) |
| Graceful Shutdown | Spring Boot 종료 시 진행 중 TX 완료 |

---

## 19. 구현 순서

### 19.1 Phase 1 — 기반 구축 (Sprint 1)

1. Spring Boot 3.x 프로젝트 초기화 + pom.xml
2. PostgreSQL 15 DDL 생성 (63개 테이블 + 트리거 + 인덱스)
3. Config 파일 (Security, JPA, MyBatis, Swagger, Jackson)
4. 공통 엔티티/DTO (ReqId, MoneyAmount, TaxRate, BaseTimeEntity)
5. 예외 처리 체계 (BaseException 계층 + GlobalExceptionHandler)
6. 공통 유틸리티 (TruncationUtil, CryptoUtil, DateTimeUtil 등)
7. 상수/코드 (SystemConstants, TaxConstants, ErrorCode, CategoryCode)
8. 메시지 파일 (messages_ko, errors_ko, validation_ko)
9. REF_* 기준정보 시드 데이터 (2018~2025 세율)
10. RBAC 인증/권한 (JWT, 4단계)

### 19.2 Phase 2 — 입력 관리 (Sprint 2)

1. API-01 (POST /requests) + M1-01~03
2. M1-04~12 카테고리별 검증 + P1-01~08 개인 검증
3. Idempotency-Key 중복 방지
4. MX-03 검증 엔진 (107개 규칙)

### 19.3 Phase 3 — 계산 엔진 (Sprint 3~4)

1. M3-PREP + M3-00~06 + P3-01~04 사전 점검
2. M4 CreditCalculator (6대 핵심 → 25개 전체, M4-10 구조조정/M4-11 연결납세 포함)
3. M5 최적 조합 (상호배제, B&B/Greedy, 최저한세, 농특세)
4. MX-01 순환참조 해결

### 19.4 Phase 4 — 보고서 + 통합 (Sprint 5)

1. M6-01~05 환급액 비교/가산금/보고서/JSON 직렬화
2. API-02~11 전체 엔드포인트
3. 감사 로그 (AuditLogService)
4. 통합 테스트 (법인 10건 + 개인 10건)

---

## 20. 용어 참조

> 상세 용어사전은 `docs/01-plan/schema.md` Section 1 참조

| 한글 | 영문 (코드) | 설명 |
|------|-----------|------|
| 경정청구 | AmendmentClaim | 과다 납부 세액의 환급 요청 |
| 법인세 | CORP | 법인 소득에 부과되는 세금 |
| 종합소득세 | INC | 개인 종합소득에 부과되는 세금 |
| req_id | Request ID | 1건 경정청구의 유일 식별자 |
| 최저한세 | MinimumTax | 최소 납부 세액 (조특법 §132) |
| 상호배제 | MutualExclusion | 동시 적용 불가 조합 규칙 (10규칙) |
| 절사 | Truncation | 금액 10원 미만 버림 (반올림 금지) |

---

## 버전 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 1.0 | 2026-02-18 | 최초 작성 — 프로젝트 구조, 기술 스택, 엔티티/DTO, 예외처리, Common & Utils, API, 계산 로직 | Design (PDCA) |
| 1.1 | 2026-02-18 | Critical 수정 — Java 17 통일, 상호배제 10규칙 상세 테이블 추가 | Design (PDCA) |
| 1.2 | 2026-02-18 | Warning 보완 — (1)SMR 엔티티 3개 추가 (2)port 패키지 설계 (3)RF_* 갱신 프로세스 (4)공식 56개 전체 목록 (5)농특세/최저한세 상세 (6)F-COM-01 (7)테스트 계획 (8)가용성 SLA (9)M4-03 Phase 2 명시 | Design (PDCA) |
| 1.3 | 2026-02-18 | GAP 보완 (프롬프트 대비) — (1)M4-10 기업구조조정 §44~47 (2)M4-11 연결납세 §76의8 (3)P4-13 전자신고 2만원 (4)고용증대 §29의7 경과규정 분기 (5)청년판단 상세산식 (6)R&D 방식선택 로직 (7)소득공제↔세율구간 최적화 (8)환급가산금 안내문구 (9)감면변경 경정청구 워크플로우 (10)사후관리 리스크 안내 | Design (PDCA) |
| 1.4 | 2026-02-18 | PDCA Iterate 정합성 수정 — (1)F-INC-01~11→12 범위 3곳(W-NEW-02) (2)REF 테이블명 통일: REF_C_EMPLOYMENT_CREDIT_RATE→REF_S_EMPLOYMENT_CREDIT, REF_S_STARTUP_EXEMPTION_RATE→REF_C_STARTUP_EXEMPTION(NEW-01,02) (3)INP_PRIOR_CREDIT→INP_S_EXISTING_DEDUCTION 2곳(NEW-07) | Design (PDCA) |
