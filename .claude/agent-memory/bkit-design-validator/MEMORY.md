# Design Validator Memory - TaxServiceENTEC-ATR

## Project Structure (Current as of 2026-02-18)
- Plan: `docs/archive/2026-02/Tax-refund/Tax-refund.plan.md` (v1.4, 21 FRs, 177 SP)
- Schema: `docs/01-plan/schema.md` (v1.0, 63 tables, 5 layers)
- Design: `docs/archive/2026-02/Tax-refund/Tax-refund.design.md` (v1.5, 2331 lines)
- Requirements: `docs/requirements-tax-refund-system.md` (852 lines, 28 US)
- Reference: `참고1번-development-design-v2.md` (83 tables, direction only)
- Validation results: `review-docs/design-validation-result.md` (v3.0, score 100/100)

## Key Numbers (All Unified Across Docs)
- Formulas: 57 (F01~F41:21 + F-INC-01~12:12 + F-COM-01:1)
- CreditCalculators: 25 (M4:12 + P4:13)
- Tables: 63 (INP:20 + SMR:7 + OUT:9 + REF:26 + AUD:1)
- Check items: 77 (CORP:40 + INC:37)
- Validation rules: 107
- REST APIs: 11
- FRs (Phase 1): 21
- User Stories: US-001~US-028 (28 total)

## Key Naming Conventions
- Table prefixes: INP_/SMR_/OUT_/REF_/AUD_ (NOT RI_/SV_/CO_/RF_/AL_ from ref doc)
- Tax type suffixes: _C_ (corp), _I_ (inc), _S_ (shared)
- Provision codes: SS24, SS29_8, SS30_4, SS6, SS7, SS10
- Entity naming: PascalCase of table name (e.g., inp_request -> InpRequest)

## Validation v3.0 Final Results (2026-02-18)
- Score: 100/100, Critical 0, Open Warnings 0
- All stale numbers resolved across all docs
- All prefix issues resolved (RF_* -> REF_* complete)
- Minor: Header version numbers lag behind version history (Info only)
- Status: Implementation Approved

## Known Info-Level Issue
- Plan header says v1.3 but version history has v1.4 entry
- Design header says v1.4 but version history has v1.5 entry
- Content is correct; only metadata needs update
