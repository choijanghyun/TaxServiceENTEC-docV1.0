# Gap Detector Agent Memory

## Project: TaxServiceENTEC-ATR (Tax Refund System)

### Key Documents
- Plan: `docs/01-plan/features/Tax-refund.plan.md` - 19 FR, 150 SP, 5 Sprints
- Schema: `docs/01-plan/schema.md` - 3-layer (INP/SMR/OUT) + REF + AUD
- Corp Prompt: `법인세-프롬프트_v2.0.md` - 40+ inspection items, 20 mandatory rules
- INC Prompt: `종합소득세-프롬프트_v2.0.md` - 37 inspection items, 28 mandatory rules

### Architecture
- Level: Enterprise (4-layer Clean Architecture)
- Stack: Java 17, Spring Boot 3.x, PostgreSQL 15, JPA+MyBatis
- Pattern: Strategy Pattern for 22 CreditCalculators, B&B/Greedy optimization

### Key Gap Findings (2026-02-18)
- Overall Match Rate: 86% (Plan vs Prompts)
- Critical gaps: RefundInterest (Phase 2 but should be Phase 1), LocalTax guidance missing
- INC-specific items underrepresented: 5 items not mapped at all
- Sprint 5 has no SP allocation (needs ~26 SP for reports + E2E + perf tests)
- Total SP likely needs 170+ vs current 150

### Domain Knowledge
- 77 inspection items: Corp 40 + INC 37
- 6 core credits/exemptions in Phase 1: SS24, SS29of8, SS6, SS7, SS10 (+SS30of4 Phase 2)
- Mutual exclusion: 10 rules with year-based branching (2024 vs 2025)
- Truncation: amounts floor to 10-won, ratios floor to 3 decimals (NEVER round)
- MinimumTax: CORP=taxBase*rate, INC=calculatedTax*35%/45% (completely different)
