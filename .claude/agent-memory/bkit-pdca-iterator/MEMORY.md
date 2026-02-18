# bkit-pdca-iterator MEMORY

## 프로젝트: TaxServiceENTEC-ATR (통합 경정청구 환급액 산출 시스템)

### 핵심 수치 (확정)
- 계산 공식: **57개** (법인 21개 + 개인 12개 + 공통 1개)
- F-INC 범위: **F-INC-01~F-INC-12** (12개)
- CreditCalculator: **25개** 서브모듈
- REF_* 기준정보 테이블: **26개**
- 전체 DB 테이블: **63개** (INP 20 + SMR 7 + OUT 9 + REF 26 + AUD 1)
- 점검항목: **77개** (법인 40 + 개인 37)
- 검증규칙: **107개**
- 스토리: **28개** (계획서 기준, 요구사항 v1.0은 27개)

### 테이블 접두어 규칙 (확정)
- INP_S_* / INP_C_* / INP_I_* (입력)
- SMR_* (중간 요약)
- OUT_* (출력)
- REF_S_* (공통 기준정보) / REF_C_* (법인 전용) / REF_I_* (개인 전용)
- AUD_* (감사 로그)
- RF_* 는 구 접두어(참고1번 초안) — 절대 사용 금지

### 주요 REF 테이블명 (v1.4 설계서 확정)
- REF_S_EMPLOYMENT_CREDIT (고용공제 단가, 공통)
- REF_C_STARTUP_EXEMPTION (창업감면율, 법인 전용)
- REF_I_STARTUP_EXEMPTION (창업감면율, 개인 전용)
- REF_C_RD_MIN_TAX_EXEMPT (R&D 최저한세 배제율)

### 주요 INP 테이블명
- INP_S_EXISTING_DEDUCTION (기존 신고 공제/감면 내역, 구: INP_PRIOR_CREDIT)

### 알려진 미해결 Warning 이슈
- NEW-08: 설계서 11.3.8절 헤더 `RF_C_RD_MIN_TAX_EXEMPT` → `REF_C_RD_MIN_TAX_EXEMPT` (오기 1곳)
- NEW-09: CreditCalculator 구현체 디렉토리 목록이 23개 열거, 25개 선언 대비 2개 미기재

### 문서 위치 (아카이브 이후)
- 계획서: docs/archive/2026-02/Tax-refund/Tax-refund.plan.md (v1.3)
- 설계서: docs/archive/2026-02/Tax-refund/Tax-refund.design.md (v1.4)
- 요구사항: docs/requirements-tax-refund-system.md (v1.0)
- 스키마: docs/01-plan/schema.md (v1.0)
