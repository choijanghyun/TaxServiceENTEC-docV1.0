> **[ARCHIVE]** 2026-02-18 아카이브. 문서 내 경로(`docs/01-plan/`, `docs/02-design/`, `docs/04-report/`, `review-docs/`)는 아카이브 전 구조 기준. 실제 위치: `docs/archive/2026-02/Tax-refund/`

# Tax-refund PDCA 완료 보고서

> **Feature**: Tax-refund — 통합 경정청구 환급액 산출 시스템
> **Project**: TaxServiceENTEC-ATR
> **Report Date**: 2026-02-18
> **PDCA Status**: **Completed** (Check 97% ≥ 90%)
> **Author**: Report Generator (PDCA)

---

## 목차

1. [Executive Summary](#1-executive-summary)
2. [PDCA 진행 이력](#2-pdca-진행-이력)
3. [Plan 요약](#3-plan-요약)
4. [Design 요약](#4-design-요약)
5. [Check (분석) 결과 요약](#5-check-분석-결과-요약)
6. [Act (개선) 요약](#6-act-개선-요약)
7. [산출물 목록](#7-산출물-목록)
8. [품질 지표 종합](#8-품질-지표-종합)
9. [잔여 항목 및 다음 단계](#9-잔여-항목-및-다음-단계)
10. [Lessons Learned](#10-lessons-learned)

---

## 1. Executive Summary

### 1.1 프로젝트 개요

세무사/세무법인을 위한 **법인세·종합소득세 통합 경정청구 환급액 극대화 시스템**의 Plan → Design 문서화 PDCA 사이클을 완료했다.

| 항목 | 내용 |
|------|------|
| 목표 | 77개 점검항목 자동화, 건당 진단 3시간 → 30분 |
| 세목 | 법인세(CORP) + 종합소득세(INC) 통합 |
| 아키텍처 | Clean Architecture 4계층, Spring Boot 3.x, Java 17, PostgreSQL 15 |
| 규모 | 63 DB 테이블, 57개 계산 공식, 107개 검증규칙, 25개 CreditCalculator, 11 REST API |

### 1.2 최종 품질 지표

| 검증 항목 | 점수 | 상태 |
|-----------|:----:|:----:|
| Plan vs 프롬프트 (계획 완전성) | **93%** | PASS |
| Design Validation (설계 완전성) | **97/100** | PASS |
| Plan ← Design (역방향 정합성) | **97/100** | PASS |
| Design vs 프롬프트 (설계 커버리지) | **97%** | PASS |
| **종합 PDCA 점수** | **96%** | **PASS** |

---

## 2. PDCA 진행 이력

### 2.1 Phase 타임라인

```
[Plan]  ✅ ──→ [Design]  ✅ ──→ [Check]  ✅ ──→ [Act]  ✅ ──→ [Report]  ✅
 v1.0→1.2       v1.0→1.3       5개 분석     1회 iterate      본 문서
```

### 2.2 상세 이력

| 날짜 | Phase | 활동 | 결과 |
|------|-------|------|------|
| 2026-02-18 | Plan | Tax-refund.plan.md v1.0 초안 작성 | FR 19개, 153 SP |
| 2026-02-18 | Plan | plan-verification (프롬프트 대비 검증) | 86% → v1.1 보완 |
| 2026-02-18 | Plan | Plan v1.1 보완 (FR-20/21 추가, Phase 2 확장) | **93%** PASS |
| 2026-02-18 | Plan | Plan v1.2 (역방향 검증 반영, 수치 정합성) | 97/100 |
| 2026-02-18 | Design | Tax-refund.design.md v1.0 초안 작성 | 20개 섹션 |
| 2026-02-18 | Check | design-validation v1.0 | 78/100 |
| 2026-02-18 | Act | 설계서 v1.1 (Critical: Java 17 통일, 상호배제) | 84/100 |
| 2026-02-18 | Check | design-validation v1.2 | 93/100 |
| 2026-02-18 | Act | 설계서 v1.2 (Warning 9건 보완) | **97/100** |
| 2026-02-18 | Check | plan-vs-design 역방향 검증 | 89 → 보완 후 **97/100** |
| 2026-02-18 | Check | gap-detection (프롬프트 vs 설계서) | 95% (초기) |
| 2026-02-18 | Check | prompt-vs-design 갭 분석 (법인세+종합소득세) | **92%** |
| 2026-02-18 | Act | 설계서 v1.3 (GAP 10건 보완) | **97%** PASS |
| 2026-02-18 | Report | 본 완료 보고서 | - |

---

## 3. Plan 요약

### 3.1 문서 정보

| 항목 | 값 |
|------|-----|
| 문서 | `docs/01-plan/features/Tax-refund.plan.md` |
| 버전 | v1.2 |
| 기능 요구사항 | FR-01 ~ FR-21 (21개) |
| 스토리 포인트 | 177 SP (Phase 1 MVP) |
| Sprint 배분 | 5 Sprint (Phase 1~4 + 개인전용) |

### 3.2 Phase 구성

| Phase | 내용 | 범위 |
|-------|------|------|
| Phase 1 MVP | 6대 핵심 공제/감면 + 상호배제 + 최적화 + 보고서 | 21개 FR, 177 SP |
| Phase 2 | 심화 공제/감면 12종 + 경과규정 + 시뮬레이션 | 확장 |
| Phase 3 | 홈택스 연동, B2C 셀프서비스, 재해손실/벤처투자 | 확장 |

### 3.3 입력 문서

| 문서 | 경로 |
|------|------|
| 요구사항 정의서 | `docs/requirements-tax-refund-system.md` |
| 법인세 프롬프트 v2.0 | `refer-to-doc/법인세-프롬프트_v2.0.md` |
| 종합소득세 프롬프트 v2.0 | `refer-to-doc/종합소득세-프롬프트_v2.0.md` |
| 개발설계서 초안 (참고) | `refer-to-doc/참고1번-development-design-v2.md` |
| 업계 리서치 | `refer-to-doc/참고2번-industry-research-tax-refund-programs.md` |
| 세법기준 입력자료 | `refer-to-doc/경정청구-입력자료-세법기준-정리.md` |
| 스키마 | `docs/01-plan/schema.md` |

---

## 4. Design 요약

### 4.1 문서 정보

| 항목 | 값 |
|------|-----|
| 문서 | `docs/02-design/features/Tax-refund.design.md` |
| 최종 버전 | v1.3 |
| 섹션 수 | 20개 + 변경이력 |
| 계산 공식 | 57개 (F01~F04, F10~F19, F30~F34, F40~F41, F-INC-01~12, F-COM-01) |
| CreditCalculator | 25개 (M4:11개 + P4:13개 + 공통인터페이스:1) |

### 4.2 핵심 설계 결정

| 결정 | 내용 | 근거 |
|------|------|------|
| Clean Architecture 4계층 | Presentation → Application → Domain → Infrastructure | 테스트 용이성, 세법 변경 시 Domain만 수정 |
| Strategy Pattern (M4) | CreditCalculator 인터페이스 + 25개 서브모듈 | 공제/감면 추가 시 OCP 준수 |
| Branch & Bound + Greedy | ≤15개 항목 B&B, >15개 Greedy fallback | 최적해 보장 vs 성능 트레이드오프 |
| INSERT ONLY (INP\_) | 원본 데이터 불변, DB 트리거로 UPDATE/DELETE 차단 | 감사 추적성, 데이터 무결성 |
| 연도별 기준정보 (REF\_) | tax\_year별 독립 행, 히스토리 보존 | 6개년(2018~2025) 세법 변경 대응 |
| 법인/개인 모듈 분리 | M계열(공통/법인) + P계열(개인전용) | 77개 점검항목의 명확한 책임 분리 |

### 4.3 버전 이력

| 버전 | 주요 변경 | 점수 |
|:----:|----------|:----:|
| v1.0 | 최초 작성 (20개 섹션) | 78/100 |
| v1.1 | Java 17 통일, 상호배제 10규칙 상세 | 84/100 |
| v1.2 | Warning 9건 보완 (SMR 엔티티, RF 갱신, 공식 56개, 테스트 등) | 97/100 |
| v1.3 | GAP 10건 보완 (구조조정/연결납세/전자신고/청년판단/R&D 방식 등) | **97%** |

---

## 5. Check (분석) 결과 요약

### 5.1 수행된 분석 5종

| # | 분석명 | 대상 | 결과 |
|---|--------|------|:----:|
| 1 | Plan vs 프롬프트 | Plan ← 법인세/종합소득세 프롬프트 | **93%** |
| 2 | Design Validation | Design 내부 완전성 | **97/100** |
| 3 | Plan ← Design | Design → Plan 역방향 정합성 | **97/100** |
| 4 | Gap Detection (초기) | 프롬프트 vs 설계서 (설계검증 중 수행) | **95%** |
| 5 | Prompt vs Design 갭 분석 | 법인세/종합소득세 프롬프트 전체 대비 설계서 | 92% → **97%** |

### 5.2 분석별 주요 발견사항

#### Plan vs 프롬프트 (93%)
- v1.0에서 FR-20(환급가산금), FR-21(지방소득세) 누락 → v1.1에서 추가
- 고용증대 경과규정(§29의7) Phase 2 명시
- 종합소득세 개인전용 12종 Phase 2에 상세 배치

#### Design Validation (97/100)
- v1.0 Critical: Java 버전 불일치(11→17), 상호배제 규칙 불완전 → v1.1에서 해소
- v1.2 Warning 9건: SMR 엔티티 누락, port 패키지 미설계, 공식 불완전 → v1.2에서 해소
- 잔여 Minor 3건: 감가상각 시부인 Phase 2 연기, SonarQube 수치 근거 미명시, 구현순서 M4 22개→25개

#### Prompt vs Design 갭 분석 (92% → 97%)
- v1.2 GAP 12건 식별 (Medium 3, Low 7, Info 2)
- v1.3에서 10건 보완 완료
- 잔여 2건은 프롬프트 "참고 항목" (Info 등급)

---

## 6. Act (개선) 요약

### 6.1 Iteration 1회 — 설계서 v1.2 → v1.3

| GAP | 심각도 | 보완 내역 |
|-----|:------:|----------|
| GAP-01 | Medium | M4-10:CorpRestructuringCalc (기업구조조정 §44~47) + §11.2.3 |
| GAP-02 | Medium | M4-11:ConsolidatedTaxCalc (연결납세 §76의8) + §11.2.4 + DB컬럼 |
| GAP-03 | Low | P4-13:EFilingCreditCalc (전자신고 2만원) + F-INC-12 |
| GAP-06 | Medium | §11.2.1 EmploymentCreditCalc 내 §29의7 경과규정 분기 |
| GAP-07 | Low | §11.2.1 YouthDetermination 4단계 산식 (병역6년/절대40세/2024개정) |
| GAP-08 | Low | §11.2.2 RdCreditCalc.selectOptimalMethod() 3단계 의사결정 |
| GAP-09 | Low | M5-02a 소득공제 ↔ 세율구간 최적화 시나리오 |
| GAP-10 | Low | M6-02 환급가산금 익금불산입/과세여부 안내문구 |
| GAP-11 | Low | M1-13 감면변경 경정청구 워크플로우 (기존 취소 + 신규 적용) |
| GAP-12 | Low | M6-04a 사후관리 리스크 안내 테이블 (§6/§24/§29의8/§10) |

### 6.2 Match Rate 변화

| | v1.2 | v1.3 | 변동 |
|--|:----:|:----:|:----:|
| 법인세 점검항목 | 90.8% | 100% | +9.2%p |
| 법인세 필수규칙 | 95.0% | 100% | +5.0%p |
| 종합소득세 점검항목 | 95.3% | 100% | +4.7%p |
| 종합소득세 필수규칙 | 92.9% | 100% | +7.1%p |
| **통합 (보수적)** | **92%** | **97%** | **+5%p** |

---

## 7. 산출물 목록

### 7.1 PDCA 문서

| # | 문서 | 경로 | 버전 |
|---|------|------|:----:|
| 1 | 계획서 | `docs/01-plan/features/Tax-refund.plan.md` | v1.2 |
| 2 | 스키마 | `docs/01-plan/schema.md` | v1.0 |
| 3 | 설계서 | `docs/02-design/features/Tax-refund.design.md` | v1.3 |
| 4 | 요구사항 | `docs/requirements-tax-refund-system.md` | Draft |
| 5 | **완료 보고서** | `docs/04-report/Tax-refund.report.md` | **v1.0** |

### 7.2 검증 보고서

| # | 보고서 | 경로 | 결과 |
|---|--------|------|:----:|
| 1 | Plan vs 프롬프트 검증 | `review-docs/plan-verification-result.md` | 93% |
| 2 | Design Validation | `review-docs/design-validation-result.md` | 97/100 |
| 3 | Plan ← Design 역방향 검증 | `review-docs/plan-vs-design-verification-result.md` | 97/100 |
| 4 | Gap Detection (초기) | `review-docs/gap-detection-result.md` | 95% |
| 5 | Prompt vs Design 갭 분석 | `review-docs/prompt-vs-design-gap-result.md` | 97% |

### 7.3 참조 문서 (refer-to-doc/)

| # | 문서 | 설명 |
|---|------|------|
| 1 | 법인세-프롬프트\_v2.0.md | 법인 40개 점검항목, 20개 필수규칙 |
| 2 | 종합소득세-프롬프트\_v2.0.md | 개인 37개 점검항목, 28개 필수규칙 |
| 3 | 참고1번-development-design-v2.md | 개발설계서 초안 (방향성 참조) |
| 4 | 참고2번-industry-research-tax-refund-programs.md | 8개 서비스 비교 분석 |
| 5 | 경정청구-입력자료-세법기준-정리.md | 공제/감면별 입력자료 세법 근거 |

---

## 8. 품질 지표 종합

### 8.1 PDCA 품질 매트릭스

| 지표 | 기준 | 실제 | 판정 |
|------|:----:|:----:|:----:|
| Plan 완전성 (프롬프트 대비) | ≥ 90% | **93%** | PASS |
| Design 완전성 (내부 일관성) | ≥ 90/100 | **97/100** | PASS |
| Plan ↔ Design 정합성 | ≥ 90/100 | **97/100** | PASS |
| Design 커버리지 (프롬프트 대비) | ≥ 90% | **97%** | PASS |
| Critical 이슈 | 0건 | **0건** | PASS |
| PDCA Iteration 횟수 | ≤ 5 | **1회** | PASS |

### 8.2 설계 규모 지표

| 지표 | 값 |
|------|-----|
| DB 테이블 | 63개 (5계층) |
| 계산 공식 | 57개 |
| 검증 규칙 | 107개 (6유형 + INC 전용 6유형) |
| CreditCalculator | 25개 (M4:11 + P4:13 + Interface:1) |
| REST API | 11개 엔드포인트 |
| 점검항목 | 77개 (법인40 + 개인37) |
| 상호배제 규칙 | 10개 (조특법 §127④) |
| 입력 카테고리 | 32개 (공통8 + CORP16 + INC8) |
| 기준정보 테이블 | 26개 (REF\_) |
| Sprint 계획 | 5 Sprint (177 SP) |

---

## 9. 잔여 항목 및 다음 단계

### 9.1 잔여 GAP (Info 등급, Phase 3 확장)

| GAP | 내용 | 예상 시점 |
|-----|------|----------|
| GAP-04 | 재해손실세액공제 | Phase 3 |
| GAP-05 | 벤처투자 소득공제 | Phase 3 |

### 9.2 보수적 보정 항목 (구현 시 정밀도 확인)

| 항목 | 보완 방안 |
|------|----------|
| 기업구조조정 적격요건 세부 (10+개) | 구현 시 체크리스트 확장 |
| 연결납세 내부거래 제거 규칙 | 구현 시 유형별 상세 설계 |
| R&D 국가전략 공제율 세부 유형 | REF 테이블 세분화 |

### 9.3 다음 단계 (PDCA 사이클 이후)

| 순서 | 활동 | 설명 |
|:----:|------|------|
| 1 | `/pdca do Tax-refund` | Phase 1 구현 시작 (Sprint 1: 기반 설정) |
| 2 | Phase 1 Sprint 배분 | §19 구현 순서에 따른 5 Sprint 진행 |
| 3 | 단위 테스트 | §17 테스트 계획 기반 (116+ 테스트 케이스) |
| 4 | `/pdca analyze Tax-refund` | 설계 vs 구현 갭 분석 (Code Level) |
| 5 | Phase 2 계획 수립 | 심화 공제/감면 12종 + 경과규정 |

---

## 10. Lessons Learned

### 10.1 효과적이었던 점

| 항목 | 상세 |
|------|------|
| **다중 검증 전략** | 5종 분석(Plan↔프롬프트, Design내부, Plan↔Design, 프롬프트↔Design, 초기Gap)으로 누락 최소화 |
| **Iterative 보완** | 설계서 4회 버전업(v1.0→1.3)을 통해 78점→97%까지 점진 개선 |
| **프롬프트 기반 검증** | 법인세/종합소득세 프롬프트를 "ground truth"로 활용하여 실무 요구사항 누락 방지 |
| **양방향 검증** | Plan→Design + Design→Plan 역방향 검증으로 불일치 조기 발견 |

### 10.2 개선이 필요한 점

| 항목 | 상세 |
|------|------|
| **구조조정/연결납세 조기 식별** | 법인 전용 특수 시나리오(§44~47, §76의8)가 초기 설계에서 누락 — 프롬프트 전수 대조를 Plan 단계에서 수행했으면 조기 발견 가능 |
| **상세 로직 깊이** | 청년판단/R&D 방식선택 등 세부 의사결정 로직이 "기능 존재" 수준에서 멈추는 경향 — 설계 시 STEP별 의사결정 트리 수준까지 명시 필요 |
| **보고서 안내 항목** | 계산 로직과 별개로 "보고서에 표시할 안내 항목"(익금불산입, 사후관리 등)이 누락되기 쉬움 — 보고서 템플릿 체크리스트 활용 권장 |

---

> **PDCA 사이클 완료 선언**
>
> Tax-refund 기능의 Plan → Design 문서화 PDCA 사이클이 **Match Rate 97%** (≥ 90% 기준 충족)으로 완료되었습니다.
> 다음 단계: 구현(Do) 진입 시 `/pdca do Tax-refund` 실행.
