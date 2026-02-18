# Tax-refund 문서 간 Gap 분석 결과서 (v2.0)

> **Project**: TaxServiceENTEC (통합 경정청구 환급액 산출 시스템)
> **분석일**: 2026-02-18
> **분석 유형**: 7축 교차 Gap 분석 (Cross-document 7-axis gap analysis)
> **대상 문서**:
>   - D1: 요구사항 (`docs/requirements-tax-refund-system.md`)
>   - D2: 계획서 (`docs/archive/2026-02/Tax-refund/Tax-refund.plan.md` v1.2)
>   - D3: 설계서 (`docs/archive/2026-02/Tax-refund/Tax-refund.design.md` v1.3)
>   - D4: 스키마 (`docs/01-plan/schema.md` v1.0)
>   - R1: 법인세 프롬프트 (`refer-to-doc/법인세-프롬프트_v2.0.md`)
>   - R2: 종합소득세 프롬프트 (`refer-to-doc/종합소득세-프롬프트_v2.0.md`)

---

## 1. 종합 Match Rate: 94%

| 검증 축 | 점수 | 상태 |
|---------|:----:|:----:|
| 요구사항 ↔ 계획서 (D1 vs D2) | 96% | PASS |
| 계획서 ↔ 설계서 (D2 vs D3) | 93% | PASS |
| 법인세 프롬프트 ↔ 설계서 (R1 vs D3) | 97% | PASS |
| 종합소득세 프롬프트 ↔ 설계서 (R2 vs D3) | 98% | PASS |
| 필수 규칙 커버리지 | 100% | PASS |
| 수치 정합성 (D2 vs D3 vs D4) | 86% | WARNING |
| **종합** | **94%** | **PASS** |

---

## 2. GAP 요약: Critical 0 / Warning 7 / Info 7 = 총 14건

### 2.1 Warning-Level GAPs (7건, 고유 5건)

| GAP ID | 항목 | 불일치 내용 | 권장 수정 |
|--------|------|-----------|----------|
| **PD-01/NUM-01** | CreditCalculator 수 | 계획서 "22개" vs 설계서 "25개" (v1.3에서 M4-10, M4-11, P4-13 추가) | 계획서 4.2절 → "25개" |
| **PD-02/NUM-02** | 계산 공식 수 | 요구사항/계획서 "56개" vs 설계서 "57개" (F-INC-12 추가) | 요구사항/계획서 → "57개" |
| **NUM-03** | 스키마 OUT 주석 | 스키마 line 223 "22개 서브모듈" vs 설계서 "25개" | 스키마 → "25개" |
| **NEW-01** | REF 고용 테이블명 | 설계서 `REF_C_EMPLOYMENT_CREDIT_RATE` vs 스키마 `REF_S_EMPLOYMENT_CREDIT` | 접두어(C/S) 및 접미어(_RATE) 통일 |
| **NEW-02** | REF 창업 테이블명 | 설계서 `REF_S_STARTUP_EXEMPTION_RATE` vs 스키마 `REF_C_STARTUP_EXEMPTION` | 접두어(S/C) 및 접미어(_RATE) 통일 |

### 2.2 Info-Level GAPs (7건)

| GAP ID | 항목 | 설명 |
|--------|------|------|
| RQ-01 | US-014 Phase 분류 | 요구사항/계획서 모두 Phase 2로 일관 |
| RQ-02 | 스토리 수 표기 | 관련문서 표에 "27개"로 기재되나 실제 US-028 포함 28개 |
| PD-03 | M1-13 감면변경 경정청구 | 설계서 v1.3 신규 추가, Plan FR-05에 암묵적으로 포함 |
| PD-04 | M6-04a 사후관리 리스크 | 설계서 v1.3 보고서 강화 추가 |
| PD-05 | M5-02a 소득공제 최적화 | 설계서 v1.3 프롬프트 갭 해소용 추가 |
| NEW-03 | 계획서 관련문서 표 | "27개 스토리" → "28개 스토리"로 갱신 필요 |
| **NEW-07** | **INP_PRIOR_CREDIT 미정의** | 설계서 M1-13에서 참조하나 스키마에 테이블 정의 없음 |

---

## 3. G-01 해결 확인

**확인 완료 (RESOLVED)**: 전자신고 세액공제(2만원)는 설계서 v1.2에서 누락(gap-detection-result.md 보고)되었으나, v1.3에서 해결됨(prompt-vs-design-gap-result.md 보고). 두 보고서의 차이는 설계서 버전 차이에 의한 것.

근거:
- design.md line 254: `EFilingCreditCalc.java  # P4-13 전자신고 세액공제 (2만원 정액)`
- design.md line 1873: `F-INC-12 | 전자신고 세액공제 | 20,000원 (정액)`

---

## 4. 우선 조치 사항

| 우선순위 | 조치 | 대상 | 영향도 |
|:--------:|------|------|--------|
| 1 | **REF 테이블명 통일** (NEW-01, NEW-02) | 설계서 12.4절 또는 스키마 7절 | 높음 — 구현 시 혼란 유발 가능 |
| 2 | **INP_PRIOR_CREDIT 정의** (NEW-07) | 스키마에 신규 테이블 추가 또는 기존 INP_C_CREDIT_CARRYFORWARD 매핑 명시 | 높음 |
| 3 | CreditCalculator 22→25개 갱신 (PD-01/NUM-01/NUM-03) | 계획서 2곳 + 스키마 1곳 | 중간 |
| 4 | 계산 공식 56→57개 갱신 (PD-02/NUM-02) | 요구사항 2곳 + 계획서 6곳 | 중간 |
| 5 | 스토리 수 27→28개 갱신 (NEW-03) | 계획서 관련문서 표 | 낮음 |

---

## 5. 변경 이력

| 버전 | 일자 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 2.0 | 2026-02-18 | 7축 전체 재검증 — (1) REF 테이블명 불일치 2건 발견 (NEW-01, NEW-02) (2) INP_PRIOR_CREDIT 미정의 발견 (NEW-07) (3) CreditCalculator/공식 수 불일치 재확인 (4) G-01 해결 확인 완료. Match Rate 94% | Gap Detection Agent |
