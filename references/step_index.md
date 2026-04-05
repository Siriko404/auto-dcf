# AutoDCF Step Index

Each step has its own spec file in `steps/`. Read ONE step at a time — never load all steps into context.

## Phase 1: Data Acquisition (Sub-Steps 0a-0g)

| Step | File | Output | Key Deliverable |
|------|------|--------|-----------------|
| 0a | `step_00a_company_id.md` | company_profile.csv | Ticker, exchange, FY end, auditor |
| 0b | `step_00b_financials.md` | financials.csv | IS/BS/CF for 4 FYs with full provenance |
| 0c | `step_00c_note_disclosures.md` | financials.csv (appended) | PP&E breakdown (excl ROU), tax rate, debt terms |
| 0d | `step_00d_peers.md` | peers.csv | 5+ peers with market data for WACC |
| 0e | `step_00e_market_data.md` | market_data.csv | Rf, ERP, CRP, size premium |
| 0f | `step_00f_assumptions.md` | assumptions.csv | 3 scenarios x 8 drivers with rationale |
| 0g | `step_00g_validation.md` | validation_report.md | BLOCKING gate: all CSVs cross-validated |

## Phase 2: Model Build (Steps 1-17)

| Step | File | Group | Key Deliverable |
|------|------|-------|-----------------|
| 1 | `step_01_skeleton.md` | inputs/ | 4 tabs, row_map.py generated, all headers |
| 2 | `step_02_drivers.md` | inputs/ | CHOOSE switch at F7, 8 driver groups |
| 3 | `step_03_wacc.md` | inputs/ | 5 peers, Hamada, CAPM with CRP, WACC |
| 4 | `step_04_other_inputs.md` | inputs/ | Dates, balances, tax rate, useful life, rates |
| 5 | `step_05_revenue.md` | model/ | Historical GAAP from CSV + forecast formulas |
| 6 | `step_06_cost.md` | model/ | COGS, SGA, Other Opex from CSV + formulas |
| 7 | `step_07_income_statement.md` | model/ | Full IS: Revenue through Net Income, EBITDA check |
| 8 | `step_08_working_capital.md` | model/ | AR (revenue-driven), Inv/AP (COGS-driven) corkscrews |
| 9 | `step_09_depreciation.md` | model/ | Dual corkscrew, half-year convention |
| 9b | `step_09b_asset_ppe.md` | model/ | PP&E summary corkscrew (12th schedule) |
| 10 | `step_10_debt.md` | model/ | Debt corkscrew, avg balance interest |
| 11 | `step_11_levered_tax.md` | model/ | Current + Deferred tax from EBT |
| 12 | `step_12_unlevered_tax.md` | model/ | Unlevered tax from EBIT, Tax Shield |
| 13 | `step_13_ufcf.md` | model/ | EBITDA + NI methods, reconciliation = 0 |
| 14 | `step_14_dcf_perpetuity.md` | dcf/ | Gordon Growth TV, end-of-year, EV bridge |
| 15 | `step_15_dcf_multiple.md` | dcf/ | EBITDA exit multiple TV |
| 16 | `step_16_outputs.md` | outputs/ | 8 sensitivity tables, scenario comparison |
| 17 | `step_17_cover.md` | outputs/ | 7 checks, conditional formatting |

## Phase 3: Integration

| Step | File | Scope | Key Deliverable |
|------|------|-------|-----------------|
| 18 | `step_18_integration_audit.md` | All | End-to-end data flow, cross-sheet, formatting |

## Supplementary Files

| File | Purpose |
|------|---------|
| `step_09_audit.md` | Gold-standard audit protocol for depreciation (most bug-prone) |
| `step_09_fixes.md` | Common depreciation fix patterns |

## Step Dependencies (forward references)

Steps 7-13 create cross-references to schedules not yet built. The auditor should flag these as `PENDING` (not `FAILED`) until the referenced step is complete:

| Step | References (forward) | Resolved After |
|------|---------------------|----------------|
| 7 (IS) | D&A -> Step 9, Interest -> Step 10, Tax -> Step 11 | Steps 9, 10, 11 |
| 13 (UFCF) | All upstream schedules | Step 12 |
| 14-15 (DCF) | UFCF -> Step 13, WACC -> Step 3 | Step 13 |
| 16 (Outputs) | All Model schedules | Step 15 |
| 17 (Cover) | Model checks -> Steps 11-13 | Step 16 |

## Agent Assignment

| Phase | Agent Type | Spawned Per |
|-------|-----------|-------------|
| Data Acquisition (0a-0g) | `autodcf-data-researcher` | Sub-step |
| Model Build (1-17) | `autodcf-builder` | Step |
| Checklist Audit | `dcf-auditor` | Step |
| Adversarial Audit | `autodcf-adversarial` | Step |
| Integration (18) | `dcf-auditor` + `autodcf-adversarial` | Once |
