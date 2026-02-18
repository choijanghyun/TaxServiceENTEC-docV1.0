# Design Validator Memory - TaxServiceENTEC

## Project Structure
- Plan: `docs/01-plan/features/Tax-refund.plan.md`
- Schema: `docs/01-plan/schema.md` (63 tables, 5 layers)
- Design: `docs/02-design/features/Tax-refund.design.md`
- Reference: `참고1번-development-design-v2.md` (83 tables, direction only)
- Validation results: `old-doc/design-validation-result.md`

## Key Naming Conventions
- Table prefixes: INP_/SMR_/OUT_/REF_/AUD_ (NOT RI_/SV_/CO_/RF_/AL_ from ref doc)
- Tax type suffixes: _C_ (corp), _I_ (inc), _S_ (shared)
- Provision codes in code: SS24, SS29_8, SS30_4, SS6, SS7, SS10
- Entity naming: PascalCase of table name (e.g., inp_request -> InpRequest)

## Critical Issue Found (2026-02-18)
- Java 1.8 in design.md conflicts with Spring Boot 3.x (requires Java 17)
- pom.xml has java.version=1.8 with spring-boot-parent 3.2.5 -- impossible combo
- Plan and reference both specify Java 17

## Schema Change from Reference
- 83 tables (ref) -> 63 tables (schema.md) -- intentional simplification
- Prefix change documented in schema.md section 11.2
