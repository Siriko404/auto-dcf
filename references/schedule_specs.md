# AutoDCF Schedule Specifications

Detailed specification for each of the 17 quantum steps. Each step defines what to build, formula patterns, acceptance criteria, and known traps.

## Notation

- `{COL}` = current column letter (F through N)
- `{PREV}` = previous column letter
- `${ROW}` = absolute row reference using the named Python constant
- `Inputs!` / `Model!` = sheet prefix (always required for cross-sheet refs)
- Blue = hardcoded input | Black = formula | Green = cross-sheet reference

---

## Step 1: Workbook Skeleton

### What to build
Create the workbook with 4 tabs, column layout, headers, freeze panes, and ALL row constants.

### Python requirements
```python
# Row constants defined INLINE using running r counter
# Each schedule section starts with current r and defines its rows:
r = 3; MODEL_REV_HEADER = r
r += 1  # blank
r += 1; REV_ROW = r
r += 1; REV_GROWTH_ROW = r
# ... pattern continues for all ~150 row constants
# NEVER use arithmetic offsets (r+1, row-2)
# ALWAYS use the named constant in formulas

HIST_COLS = ['F', 'G', 'H', 'I']
FCST_COLS = ['J', 'K', 'L', 'M', 'N']
ALL_COLS = HIST_COLS + FCST_COLS
```

### Tab structure
1. Cover (leftmost) > 2. Outputs > 3. Inputs > 4. Model

### Column widths (all tabs)
A=2.5, B=4, C=35, D=8, E=14, F-N=14 each

### Freeze panes
Model: E2, Inputs: E2

### Year headers
Row 1 or 2: `2022A, 2023A, 2024A, 2025A, 2026F, 2027F, 2028F, 2029F, 2030F` (adjust to actual company FY)

### Acceptance criteria
- [ ] 4 tabs in correct order: Cover, Outputs, Inputs, Model
- [ ] Column A width ~2.5 on all data tabs
- [ ] Columns F-N width ~14
- [ ] Freeze panes at E2 on Model and Inputs
- [ ] Row constants defined inline with running r counter (~150 total across 12 schedules)
- [ ] Year headers present with A/F suffixes
- [ ] 12 schedule header rows present in Model (bold, bottom border)
- [ ] No formulas yet — skeleton only
- [ ] HIST_COLS and FCST_COLS defined

### Known traps
- Not defining row constants upfront (forces arithmetic offsets later)
- Wrong tab order

---

## Step 2: Drivers Section (Inputs Sheet)

### What to build
Rows 1-42 of Inputs. Three-scenario driver switch with CHOOSE formulas.

### Structure
```
Row 1:  "OPERATING ASSUMPTIONS" section header
Row 3:  "Driver Switch"
Row 7:  Active Scenario value (F7, data validation: 1,2,3, default=2)
Row 9:  "Revenue Drivers" subsection
Row 10: Volume Growth - Best (blue)
Row 11: Volume Growth - Base (blue)
Row 12: Volume Growth - Worst (blue)
Row 13: Volume Growth - Active = CHOOSE($F$7, F10, F11, F12)
Row 15: Pricing - Best/Base/Worst/Active (same pattern)
Row 19-22: COGS % — Best/Base/Worst/Active
Row 24-27: SGA % — Best/Base/Worst/Active
Row 29-32: Other Opex — Best/Base/Worst/Active
Row 34-37: Capex % — Best/Base/Worst/Active
Row 39-42: AR Days — Best/Base/Worst/Active
Row 44-47: Inventory Days — Best/Base/Worst/Active
Row 49-52: AP Days — Best/Base/Worst/Active
```

### Formula pattern (Active rows)
`=CHOOSE(Inputs!$F$7, {COL}{BEST_ROW}, {COL}{BASE_ROW}, {COL}{WORST_ROW})`
- $F$7 is ABSOLUTE (dollar signs on both row and column)
- Each forecast column (J-N) has its own CHOOSE formula

### Acceptance criteria
- [ ] F7 has data validation (list: 1,2,3)
- [ ] F7 default value is 2 (Base Case)
- [ ] Every Active row uses `CHOOSE($F$7, best, base, worst)`
- [ ] CHOOSE references correct scenario rows (not off-by-one)
- [ ] At least 7 driver groups: Vol Growth, Pricing, COGS%, SGA%, Capex%, AR Days, Inv Days, AP Days
Note: COGS% and SGA% may alternatively be per-year direct blue inputs in Other Inputs (rows 116+) rather than CHOOSE-switched scenarios, depending on how granular the user wants scenario control.
- [ ] All scenario inputs have blue font (#0000CC)
- [ ] All CHOOSE cells have black font (#000000)
- [ ] Percentages formatted "0.0%"
- [ ] Values are reasonable vs research file ratios
- [ ] $F$7 is absolute in all CHOOSE formulas

### Known traps
- CHOOSE referencing wrong scenario rows (off-by-one)
- Missing $ on F7 reference (becomes relative, breaks when copied)
- Not including data validation on F7

---

## Step 3: WACC Section (Inputs Sheet)

### What to build
Rows 45-90. Peer comps, Hamada beta, CAPM, WACC.

### Structure
```
Row 45: "WACC CALCULATION" header
Row 47: "Comparable Companies"
Row 48: Column headers
Row 49-53: 5 peers (Market Cap, Debt, Cash, D/E, Levered Beta, Tax Rate, Unlevered Beta)
Row 55: Average Unlevered Beta = AVERAGE formula
Row 56: Median Unlevered Beta = MEDIAN formula
Row 58: "Beta Calculation"
Row 59: Selected Unlevered Beta (blue input)
Row 60: Target D/E (blue)
Row 61: Marginal Tax Rate (blue)
Row 62: Re-levered Beta = B_U * (1 + (1-T)*D/E)
Row 64: "Cost of Equity (CAPM)"
Row 65: Risk-Free Rate Rf (blue)
Row 66: Equity Risk Premium ERP (blue)
Row 67: Country Risk Premium CRP (blue)
Row 68: Size Premium (blue, 0 if not used)
Row 69: Beta (green -> Row 62)
Row 70: Cost of Equity Re = Rf + CRP + ERP*Beta + Size
Row 72: "Cost of Debt"
Row 73: Rf (green -> Row 65)
Row 74: Credit Spread (blue)
Row 75: Pre-tax Kd = Rf + Spread
Row 76: Tax Rate (green -> Row 61)
Row 77: After-tax Kd = Kd * (1-T)
Row 79: "WACC"
Row 80: Equity Weight We (blue)
Row 81: Debt Weight Wd = 1 - We
Row 82: Re (green -> Row 70)
Row 83: Rd(1-T) (green -> Row 77)
Row 84: WACC = We*Re + Wd*Rd(1-T)
```

### Key formulas
- Hamada unlevering: `=F{lev_beta}/(1+(1-F{tax})*F{de})` per peer
- Hamada re-levering: `=F{unlev_beta}*(1+(1-F{tax})*F{target_de})`
- CAPM: `=F{rf}+F{crp}+F{erp}*F{beta}+F{size}`
- After-tax Kd: `=F{pretax_kd}*(1-F{tax})`
- WACC: `=F{we}*F{re}+F{wd}*F{aftertax_kd}`
- AVERAGE/MEDIAN: Excel formulas, NOT hardcoded values

### Acceptance criteria
- [ ] 5 real peer companies with sourced data (not fabricated)
- [ ] Hamada unlevering: B_U = B_L / (1 + (1-T) * D/E)
- [ ] Both AVERAGE and MEDIAN unlevered beta via Excel formulas
- [ ] Re-levering: B_L = B_U * (1 + (1-T) * D/E)
- [ ] CAPM includes CRP term
- [ ] Credit spread from actual facility terms
- [ ] WACC = We*Re + Wd*Rd(1-T)
- [ ] All peer data blue font with source comments
- [ ] All formulas black/green font (green if cross-referencing within Inputs)
- [ ] Peer statistics use Excel AVERAGE/MEDIAN (not Python-computed)
- [ ] Every blue input has a cell comment with source

### Known traps
- Fabricated peer data (#9)
- Hardcoded AVERAGE/MEDIAN (#15)
- Wrong Hamada direction (unlevering vs re-levering)
- Missing CRP in CAPM

---

## Step 4: Other Inputs (Inputs Sheet)

### What to build
Rows 92-136. Dates, opening balances, rates, per-year assumptions.

### Structure
```
Row 92:  "OTHER MODEL INPUTS" header
Row 94:  "Key Dates"
Row 95:  Valuation Date (blue)
Row 96:  Cash Flow Dates per forecast column (blue, one per col J-N)
Row 98:  "Opening Balances"
Row 99:  PP&E NBV (blue, from most recent BS)
Row 100: Accounts Receivable (blue)
Row 101: Inventory (blue)
Row 102: Accounts Payable (blue)
Row 103: Total Debt (blue)
Row 104: Cash & Equivalents (blue)
Row 105: Levered Tax Loss CF (blue, 0 if none)
Row 106: Unlevered Tax Loss CF (blue, 0 if none)
Row 108: "Rates & Assumptions"
Row 109: Statutory Tax Rate (blue, from tax notes)
Row 110: Useful Life PP&E in years (blue)
Row 111: Accounting Depreciation Rate (blue, = 1/Useful Life or company-specific)
Row 112: Tax Depreciation Rate / CCA Rate (blue, from tax notes)
Row 113: Debt Interest Rate (blue, from facility terms)
Row 114: Terminal Growth Rate (blue)
Row 115: Exit EBITDA Multiple (blue)
Row 117: "Year-by-Year Assumptions" (for fine-tuning beyond drivers)
Row 118-121: Shares Outstanding per year (blue)
```

### Acceptance criteria
- [ ] Every opening balance matches most recent audited BS
- [ ] Statutory tax rate from tax notes (NOT effective rate)
- [ ] Useful life is in YEARS and reasonable (e.g., 5-20, NOT a percentage like 26.5%)
- [ ] Debt interest rate from actual credit facility terms
- [ ] Cash is a real balance (not hardcoded in a formula elsewhere)
- [ ] All blue inputs have source comments
- [ ] Every number has a corresponding entry in the research file
- [ ] Useful life row and tax rate row are clearly separated and labeled
- [ ] Accounting depreciation rate and tax/CCA depreciation rate both present and sourced

### Known traps
- Wrong PP&E value (#8: $85M estimate vs $44.8M actual)
- Tax rate mix-up (effective vs statutory)
- Useful life and tax rate on adjacent rows — LABEL THEM CLEARLY
- Cash not in input cell (hardcoded later in DCF bridge)

---

## Step 5: Revenue Schedule (Model Sheet)

### What to build
First schedule in Model. Historical GAAP revenue + forecast.

### Structure
```
Row {hdr}: "REVENUE SCHEDULE" header (bold, bottom border)
Row {rev}: Revenue
Row {vol}: Volume Growth % (green -> Inputs active driver)
Row {price}: Pricing Growth % (green -> Inputs active driver)
Row {growth}: Revenue Growth YoY %
```

### Formula patterns
- Historical Revenue (F-I): hardcoded GAAP value, BLUE font
- Forecast Revenue (J-N): `={PREV}{REV}*(1+Inputs!{COL}${VOL_ACTIVE})*(1+Inputs!{COL}${PRICE_ACTIVE})`
- Vol/Price display: `=Inputs!{COL}${ACTIVE_ROW}` (GREEN font)
- YoY Growth: `={COL}{REV}/{PREV}{REV}-1` (BLACK font)

### Acceptance criteria
- [ ] Historical revenue matches GAAP from research file
- [ ] Historical cells blue font
- [ ] Forecast = Prior * (1+Vol) * (1+Price)
- [ ] Links to ACTIVE (CHOOSE output) driver rows, not scenario rows
- [ ] Cross-sheet refs green font
- [ ] First forecast references last historical correctly
- [ ] Growth % row computes correctly

### Known traps
- Linking to scenario row instead of CHOOSE active row
- Missing $ on Inputs row reference

---

## Step 6: Cost Schedule (Model Sheet)

### What to build
COGS, SGA, Other Operating Expenses.

### Structure
```
Row {hdr}: "COST SCHEDULE" header
Row {cogs}: COGS
Row {cogs_pct}: COGS as % of Revenue
Row {sga}: SGA
Row {sga_pct}: SGA %
Row {other}: Other Opex
Row {other_pct}: Other Opex %
Row {total}: Total Operating Costs (double border)
```

### Formula patterns
- Historical (F-I): GAAP values, BLUE font
- Forecast COGS: `=-{COL}${REV}*Inputs!{COL}${COGS_ACTIVE}` (stored negative)
  OR `={COL}${REV}*Inputs!{COL}${COGS_ACTIVE}` with explicit sign convention
- Pct rows: `=-{COL}{COGS}/{COL}{REV}` (display as positive %)

### Acceptance criteria
- [ ] Historical costs match GAAP
- [ ] Forecast linked to Revenue * % from Inputs ACTIVE row
- [ ] Sign convention consistent across all cost lines
- [ ] Total = sum of all cost items
- [ ] Percentage rows compute correctly
- [ ] Font colors correct (blue hist, black formula, green cross-sheet)

### Known traps
- Sign convention inconsistency (costs positive vs negative)
- Wrong Inputs row for COGS% vs SGA%

---

## Step 7: Income Statement (Model Sheet)

### What to build
Full IS from Revenue through Net Income.

### Structure
```
Row {hdr}: "INCOME STATEMENT" header
Row {rev}: Revenue (green -> Rev Schedule)
Row {cogs}: COGS (green -> Cost Schedule)
Row {gross}: Gross Profit
Row {sga}: SGA (green -> Cost Schedule)
Row {other}: Other Opex (green -> Cost Schedule)
Row {da}: D&A (green -> Depreciation Schedule, Step 9)
Row {ebitda}: EBITDA (double border)
Row {ebitda_chk}: EBITDA Check (Rev-COGS-SGA-Other, verify = EBITDA)
Row {ebit}: EBIT = EBITDA - D&A
Row {interest}: Interest (green -> Debt Schedule, Step 10)
Row {ebt}: EBT = EBIT - Interest
Row {tax}: Tax (green -> Levered Tax Schedule, Step 11)
Row {ni}: Net Income (double border)
```

### Formula patterns
- IS Revenue: `={COL}${REV_SCHEDULE_ROW}` (green, pulls from Revenue Schedule above)
- Gross: `={COL}{IS_REV}+{COL}{IS_COGS}` (COGS negative) or `={COL}{IS_REV}-{COL}{IS_COGS}`
- EBITDA: `={COL}{GROSS}+{COL}{SGA}+{COL}{OTHER}` (costs negative)
- EBITDA Check: `={COL}{IS_REV}+{COL}{IS_COGS}+{COL}{IS_SGA}+{COL}{IS_OTHER}`
- D&A, Interest, Tax: green references to their schedules (may be forward references until those steps are built)

### Historical handling
- ALL historical IS rows: hardcoded GAAP (blue font)
- EBITDA Check should tie even for historicals
- Historical D&A, Interest, Tax from actual IS

### Acceptance criteria
- [ ] Historical values match GAAP IS from research
- [ ] Revenue/COGS/SGA pull from their schedules (not re-hardcoded in forecast)
- [ ] EBITDA = Gross - SGA - Other (or equivalent)
- [ ] EBITDA Check ties to EBITDA (difference = 0)
- [ ] EBIT = EBITDA - D&A
- [ ] EBT = EBIT - Interest
- [ ] Net Income = EBT - Tax
- [ ] D&A/Interest/Tax reference their schedules (forward refs OK)
- [ ] Double border on EBITDA and Net Income
- [ ] Historical blue, formula black, cross-ref green

### Known traps
- "Adjusted" EBITDA (#7)
- Historical IS overwritten by later steps (#13)

---

## Step 8: Working Capital Schedule (Model Sheet)

### What to build
AR, Inventory, AP corkscrews + NWC + Change in NWC.

### Corkscrew pattern (for each component)
```
Opening = Prior year Closing
Change = Closing - Opening
AR Closing = Revenue * AR_Days / 365 (revenue-driven)
Inventory Closing = COGS * Inv_Days / 365 (COGS-driven, NOT revenue)
AP Closing = COGS * AP_Days / 365 (COGS-driven)
```

### Structure
```
Row {hdr}: "WORKING CAPITAL SCHEDULE" header
Row {ar_o}: AR Opening
Row {ar_ch}: AR Change
Row {ar_c}: AR Closing
Row {inv_o}: Inventory Opening
Row {inv_ch}: Inventory Change
Row {inv_c}: Inventory Closing
Row {ap_o}: AP Opening
Row {ap_ch}: AP Change
Row {ap_c}: AP Closing
Row {nwc}: NWC = AR + Inv - AP
Row {nwc_pct}: NWC % of Revenue
Row {delta}: Change in NWC = Current - Prior (double border)
```

### Acceptance criteria
- [ ] Corkscrew: Opening = Prior Closing for all 3 components
- [ ] First forecast Opening = last historical Closing
- [ ] Historical balances match actual BS
- [ ] AR Days, Inv Days, AP Days based on ACTUAL historical ratios. Inventory and AP are COGS-driven, NOT revenue-driven.
- [ ] AR Closing = Revenue * AR_Days / 365; Inv Closing = COGS * Inv_Days / 365; AP Closing = COGS * AP_Days / 365
- [ ] Change in NWC computed correctly
- [ ] Signs: AR/Inv increase = cash outflow, AP increase = cash inflow
- [ ] Double border on Change in NWC

### Known traps
- NWC% too low (#5: 16% vs actual 22%)
- Opening/Closing confusion (like #2)

---

## Step 9: Depreciation & PP&E Schedule (Model Sheet)

### What to build
Multi-vintage PP&E corkscrew. THE most bug-prone schedule.

### Structure
```
Row {hdr}: "DEPRECIATION SCHEDULE" header
--- Existing Assets Sub-Corkscrew ---
Row {ex_open}: Existing Assets Opening NBV
Row {ex_depr}: Existing Assets Depreciation = -Opening * Accounting Depr Rate
Row {ex_close}: Existing Assets Closing = Opening + Depreciation
--- New Assets Sub-Corkscrew ---
Row {new_open}: New Assets Opening NBV (= prior year New Close)
Row {new_capex}: Capex (Revenue * Capex% or from Inputs)
Row {new_depr}: New Assets Depreciation = -Opening * Rate - Capex * Rate * 0.5 (half-year convention on current capex)
Row {new_close}: New Assets Closing = Opening + Capex + Depreciation
--- Summary ---
Row {depr_tot}: Total Depreciation = Existing Depr + New Depr (double border)
Row {da_other}: Other D&A (ROU, Intangibles — flat or from Inputs)
Row {da_total}: Total D&A = Total Depr + Other D&A
```

### CRITICAL formulas
- Existing Opening: `={PREV}${EX_CLOSE}` — MUST reference Existing CLOSING, NOT depreciation
- Existing Depr: `=-{COL}${EX_OPEN}*Inputs!$F${ACCT_DEPR_RATE}` (accounting rate, NOT tax rate)
- New Opening: `={PREV}${NEW_CLOSE}` (first year = 0)
- New Capex: `={COL}${REV_ROW}*Inputs!{COL}${CAPEX_ACTIVE}` (or from Inputs directly)
- New Depr: `=-{COL}${NEW_OPEN}*Inputs!$F${ACCT_DEPR_RATE}-{COL}${NEW_CAPEX}*Inputs!$F${ACCT_DEPR_RATE}*0.5`
  HALF-YEAR CONVENTION: Current-year capex gets 50% depreciation. Prior-year NBV gets full year.
- Total Depr: `={COL}${EX_DEPR}+{COL}${NEW_DEPR}`
- Closing: each sub-corkscrew sums its components

### Acceptance criteria
- [ ] Two separate sub-corkscrews: Existing Assets and New Assets
- [ ] Opening references CLOSING row (not depreciation, not opening itself)
- [ ] Depreciation rate is ACCOUNTING rate from Inputs, NOT tax/CCA rate
- [ ] New asset depreciation uses half-year convention on current capex (Rate * 0.5)
- [ ] Capex sign: positive addition to NBV
- [ ] Depreciation sign: negative (reduces NBV)
- [ ] Closing = Opening + Capex + Depreciation (per sub-corkscrew)
- [ ] First forecast Existing Opening = last historical Closing NBV
- [ ] Historical PP&E matches actual BS
- [ ] New asset depreciation accumulates year-over-year
- [ ] Total D&A links to IS (Step 7)
- [ ] Useful life value is reasonable (5-20 years, NOT 0.265)

### Known traps
- **#1 CATASTROPHIC: Depreciation / Tax Rate instead of / Useful Life**
- **#2 CATASTROPHIC: Opening = Depreciation instead of Closing**
- Half-year convention missing on new capex (full year in year 1 overstates depreciation)
- New assets not accumulating depreciation

---

## Step 10: Debt Schedule (Model Sheet)

### What to build
Debt corkscrew + interest calculation.

### Structure
```
Row {hdr}: "DEBT SCHEDULE" header
Row {open}: Debt Opening
Row {draws}: Draws/Proceeds
Row {repay}: Repayments
Row {close}: Closing = Open + Draws - Repay (double border)
Row {rate}: Interest Rate (green -> Inputs)
Row {interest}: Interest = Opening * Rate
```

### Acceptance criteria
- [ ] Corkscrew: Opening = Prior Closing
- [ ] First forecast Opening = last historical Closing debt
- [ ] Historical debt matches BS
- [ ] Interest = Opening * Rate (not Closing)
- [ ] Rate from actual facility terms
- [ ] Interest links to IS
- [ ] **FORMULAS ONLY IN FORECAST COLUMNS** — historical IS interest NOT overwritten
- [ ] Closing = Opening + Draws - Repayments

### Known traps
- **#13: Historical IS overwritten** — formula loop must use FCST_COLS only
- Interest on closing balance instead of opening

---

## Step 11: Levered Tax Schedule (Model Sheet)

### What to build
Tax from EBT with loss carryforward.

### Structure
```
Row {hdr}: "TAX SCHEDULE — LEVERED" header
Row {ebt}: EBT (green -> IS)
--- Current Tax ---
Row {tl_open}: Tax Loss CF Opening
Row {tl_used}: Tax Loss Used = min(Opening, max(0, EBT))
Row {taxable}: Taxable Income = max(0, EBT) - Tax Loss Used
Row {current_tax}: Current Tax = Taxable * Statutory Rate
Row {tl_close}: Tax Loss CF Closing = Opening - Used + new losses
--- Deferred Tax ---
Row {acct_depr}: Accounting Depreciation (green -> Depreciation Schedule Total Depr)
Row {tax_depr}: Tax Depreciation = Opening UCC * CCA Rate (from Inputs)
Row {timing}: Timing Difference = Accounting Depr - Tax Depr
Row {deferred}: Deferred Tax = Timing Difference * Statutory Rate
--- Total ---
Row {total_tax}: Total Tax = Current + Deferred (double border)
```

### Acceptance criteria
- [ ] Levered tax from EBT (includes interest deduction)
- [ ] Statutory tax rate used (from Inputs), not effective
- [ ] Tax losses reduce taxable income correctly
- [ ] Tax cannot be negative (floor at 0)
- [ ] Loss carryforward corkscrew: Opening = Prior Closing
- [ ] Both current AND deferred tax computed
- [ ] Deferred tax from timing difference (accounting depr vs tax/CCA depr)
- [ ] Total Tax = Current + Deferred flows to IS
- [ ] Historical matches actual current tax from filing

### Known traps
- Using effective rate instead of statutory
- No floor on taxable income (negative tax)

---

## Step 12: Unlevered Tax + Tax Shield (Model Sheet)

### What to build
Tax from EBIT (no interest deduction) + Tax Shield.

### Structure
```
Row {hdr}: "TAX SCHEDULE — UNLEVERED" header
Row {ebit}: EBIT (green -> IS)
--- Current Tax ---
Row {tl_open}: Unlevered Tax Loss Opening
Row {tl_used}: Unlevered Tax Loss Used = min(Opening, max(0, EBIT))
Row {taxable}: Unlevered Taxable Income = max(0, EBIT) - Tax Loss Used
Row {current_tax}: Unlevered Current Tax = Taxable * Rate
Row {tl_close}: Unlevered Tax Loss Closing
--- Deferred Tax ---
Row {acct_depr}: Accounting Depreciation (green -> Depreciation Schedule Total Depr)
Row {tax_depr}: Tax Depreciation = Opening UCC * CCA Rate (from Inputs)
Row {timing}: Timing Difference = Accounting Depr - Tax Depr
Row {deferred}: Unlevered Deferred Tax = Timing Difference * Statutory Rate
--- Total ---
Row {ulev_total}: Unlevered Total Tax = Current + Deferred
Row {shield}: Tax Shield = Unlevered Tax - Levered Tax (positive: tax savings from debt)
```

Tax Shield is POSITIVE because levered firm pays LESS tax (interest deduction reduces taxable income). Direction: Unlevered minus Levered.

### Acceptance criteria
- [ ] Unlevered tax from EBIT (excludes interest)
- [ ] Both current AND deferred tax computed (same structure as Levered)
- [ ] Tax Shield = Unlevered - Levered (positive = tax savings from debt)
- [ ] Same statutory tax rate as levered schedule
- [ ] Loss CF corkscrew correct
- [ ] Unlevered tax flows to UFCF
- [ ] Tax Shield sign makes sense (Unlevered > Levered since interest reduces levered base)

### Known traps
- Tax shield reversed
- Using different rates for levered vs unlevered

---

## Step 13: UFCF Dual Method (Model Sheet)

### What to build
EBITDA method + NI method + reconciliation.

### Structure
```
Row {hdr}: "UNLEVERED FREE CASH FLOW" header
Row {m1}: "Method 1: EBITDA Approach"
Row {ebitda}: EBITDA (green -> IS)
Row {ulev_tax}: Unlevered Tax (green -> Unlev Tax, negative)
Row {capex}: Capex (green -> PP&E, negative)
Row {dnwc}: Change in NWC (green -> WC, negative if increase)
Row {ufcf1}: UFCF Method 1 (double border)

Row {m2}: "Method 2: Net Income Approach"
Row {ni}: Net Income (green -> IS)
Row {da}: D&A (green -> Depr, positive add-back)
Row {deftax}: Deferred Tax (green -> Levered Tax Schedule, positive add-back)
Row {int_exp}: Interest Expense (green -> Debt Schedule, positive)
Row {tax_shield}: Less: Tax Shield (green -> Unlevered Tax, negative)
Row {capex2}: Capex (green, negative)
Row {dnwc2}: Change in NWC (green, negative if increase)
Row {ufcf2}: UFCF Method 2 (double border)

Row {recon}: Reconciliation = Method 1 - Method 2 (MUST = 0)
```

### Key formulas
- UFCF1: `={EBITDA}+{ULEV_TAX}+{CAPEX}+{DNWC}` (tax, capex, NWC are negative)
- UFCF2: `={NI}+{DA}+{DEFERRED_TAX}+{INTEREST}-{TAX_SHIELD}+{CAPEX}+{DNWC}`
- Recon: `={UFCF1}-{UFCF2}` (should be 0 or < $0.01)

### Acceptance criteria
- [ ] EBITDA method: EBITDA - Unlevered Tax - Capex - ΔNWC
- [ ] NI method: NI + D&A + Deferred Tax + Interest - Tax Shield - Capex - ΔNWC
- [ ] Reconciliation = 0 (or < $0.01)
- [ ] All components link to correct schedules (green font)
- [ ] Double borders on both UFCF totals
- [ ] Decomposed form used: Interest and Tax Shield as separate line items (not Interest*(1-T) shortcut)

The Interest*(1-T) shortcut is equivalent to (+Interest - Tax Shield) only when there are no tax losses or deferred tax. The decomposed form is required for accurate reconciliation.

### Known traps
- Missing deferred tax add-back in NI method
- Wrong sign on ΔNWC
- Reconciliation off because of rounding or missing component
- Using Interest*(1-T) shortcut instead of decomposed form when deferred tax exists

---

## Step 14: DCF — Perpetuity Method (Model Sheet)

### What to build
Gordon Growth terminal value, discounting, EV bridge.

### Structure
```
Row {hdr}: "DCF — PERPETUITY GROWTH METHOD" header
Row {dates}: Cash Flow Dates (green -> Inputs)
Row {periods}: Discount Periods = (CF Date - Val Date)/365.25
Row {ufcf}: UFCF (green -> UFCF Method 1)
Row {df}: Discount Factor = 1/(1+WACC)^period
Row {pv}: PV of FCF = UFCF * DF
Row {tv_hdr}: "Terminal Value"
Row {last_fcf}: Last FCF (green -> last forecast UFCF)
Row {tgr}: TGR (green -> Inputs)
Row {wacc}: WACC (green -> Inputs)
Row {tv}: TV = Last FCF * (1+TGR) / (WACC - TGR)
Row {tv_period}: TV Period = integer N (END-of-year, NOT mid-year)
Row {pv_tv}: PV of TV = TV / (1+WACC)^N
Row {ev_hdr}: "Enterprise Value Bridge"
Row {sum_pv}: Sum of PV FCFs
Row {pv_tv2}: PV of TV
Row {ev}: EV = Sum + PV TV
Row {debt}: Less Debt (green -> Inputs/Debt closing)
Row {cash}: Plus Cash (green -> Inputs)
Row {equity}: Equity Value
Row {shares}: Shares (green -> Inputs)
Row {price}: Per Share (double border)
```

### CRITICAL: Discount period convention
- Discrete FCFs: mid-year (0.5, 1.5, 2.5, 3.5, 4.5)
- Terminal value: END-of-year integer (5.0, NOT 4.5)
- Period formula: `=(Inputs!{COL}${CF_DATE}-Inputs!$F${VAL_DATE})/365.25`

**Mid-year date convention**: Cash flow dates should be set to July 1 (or June 30) of each forecast year, representing the mid-point of the annual cash flow assumption.

### Acceptance criteria
- [ ] Periods are mid-year for FCFs (fractional)
- [ ] TV period is END-of-year (integer, e.g., 5.0)
- [ ] TV = FCF_last * (1+g) / (WACC-g) — Gordon Growth
- [ ] TGR < WACC (sanity)
- [ ] EV = Sum PV FCFs + PV TV
- [ ] Equity = EV - Debt + Cash
- [ ] Debt and Cash from INPUT cells (not hardcoded in bridge)
- [ ] Per Share = Equity / Shares
- [ ] WACC from Inputs (green)
- [ ] Double border on Per Share

### Known traps
- **#6: TV at mid-year** — must be end-of-year
- Cash hardcoded in bridge
- Shares inconsistency (#10)

---

## Step 15: DCF — Multiple Method (Model Sheet)

### What to build
Same structure as Step 14 but TV = EBITDA * Exit Multiple.

### Key difference
- TV: `={LAST_COL}${IS_EBITDA}*Inputs!$F${EXIT_MULT}` (EBITDA, not FCF)
- Everything else identical to Perpetuity method

### Acceptance criteria
- [ ] Same PV of FCFs as Perpetuity method
- [ ] TV = EBITDA * Multiple (not FCF * Multiple)
- [ ] Exit multiple from Inputs
- [ ] End-of-year TV discounting
- [ ] EV bridge same structure
- [ ] Per Share different from Perpetuity (expected)

### Known traps
- Using FCF instead of EBITDA for exit multiple
- Inconsistent discount periods vs Perpetuity method

---

## Step 16: Outputs Dashboard (Outputs Sheet)

### What to build
Two dashboards with sensitivity tables and scenario comparison.

### Dashboard structure (repeat for Perpetuity and Multiple)
```
Title row
Scenario label = CHOOSE(Inputs!$F$7, "Best Case", "Base Case", "Worst Case")
Key metrics (WACC, TGR/Multiple, EV, Per Share) — green refs to Model
EV Bridge — green refs to Model
Sensitivity Tables (4 per dashboard)
Scenario Comparison (Best/Base/Worst per share)
```

### Sensitivity table requirements
- ODD dimensions (5x5 or 7x7) — center = base case
- Center cell highlighted (distinct fill + bold)
- **EACH cell re-discounts ALL FCFs at the scenario WACC:**
```
=(SUM(Model!{FCF1}/(1+{wacc_axis})^{p1}, Model!{FCF2}/(1+{wacc_axis})^{p2}, ...)
  + TV/(1+{wacc_axis})^{tv_period} - debt + cash) / shares
```
- This is NOT `=base_EV * (base_WACC/new_WACC)` — that is linear scaling and WRONG

### Sensitivity formula construction
Each sensitivity cell formula must be built PROGRAMMATICALLY in Python using string concatenation. These formulas are 300+ characters and cannot be typed manually. Pattern:
```python
terms = []
for i, col in enumerate(FCST_COLS):
    terms.append(f"Model!{col}${UFCF_ROW}/(1+{wacc_cell})^Model!{col}${PERIOD_ROW}")
tv_term = f"Model!{tv_col}${TV_ROW}/(1+{wacc_cell})^Model!{tv_col}${TV_PERIOD_ROW}"
formula = f"=({'+'.join(terms)}+{tv_term}-{debt_ref}+{cash_ref})/{shares_ref}"
```

### Scenario comparison
- Best/Worst use CUMULATIVE compound differentials:
```
For year t: factor(t) = PRODUCT(k=1..t, (1+g_scen(k))/(1+g_base(k)))
Scenario FCF(t) = Base FCF(t) * factor(t)
```
- NOT: `Base * (1+delta)` per year (single-year scaling)

### Acceptance criteria
- [ ] Two dashboards (Perpetuity + Multiple)
- [ ] Scenario label uses CHOOSE (text, not number)
- [ ] Sensitivity ODD dimensions, center = base case
- [ ] Every sensitivity cell re-discounts each FCF
- [ ] NO linear WACC ratio scaling
- [ ] Center cell highlighted
- [ ] Best/Worst cumulative compound differentials
- [ ] EV bridge matches Model
- [ ] Green font on cross-sheet refs
- [ ] Axis labels reasonable (WACC +/-200bps, TGR +/-100bps)

### Known traps
- **#4: Linear scaling** — must re-discount
- **#14: Number not text** — CHOOSE for label
- **#3: Single-year scaling** — must be cumulative
- #15: Hardcoded statistics

---

## Step 17: Cover Sheet + Model Checks

### What to build
Model metadata and 7 automated checks.

### The 7 checks
```
1. Valuation Date populated: =IF(Inputs!F{val_date}<>"", "PASS", "FAIL")
2. CF Date populated: =IF(Inputs!J{cf_date}<>"", "PASS", "FAIL")
3. TGR < WACC: =IF(Inputs!F{tgr}<Inputs!F{wacc}, "PASS", "FAIL")
4. Debt capacity: =IF(Model!N{debt_close}>0, "PASS", "REVIEW")
5. No unused levered tax losses: =IF(Model!N{lev_tl_close}<=0, "PASS", "FLAG")
6. No unused unlevered tax losses: =IF(Model!N{ulev_tl_close}<=0, "PASS", "FLAG")
7. UFCF reconciliation: =IF(ABS(Model!N{recon})<0.01, "PASS", "FAIL")
```

### Conditional formatting
- "PASS": green fill #C6EFCE, dark green font #006100
- "FAIL": red fill #FFC7CE, dark red font #9C0006
- "FLAG"/"REVIEW": yellow fill #FFEB9C, dark yellow font #9C6500

### Acceptance criteria
- [ ] All 7 checks present
- [ ] Formulas reference Model! and Inputs! WITH sheet prefix
- [ ] Conditional formatting applied
- [ ] Company name and metadata present
- [ ] All checks evaluate to PASS for a correct model
- [ ] Check 7 uses ABS < threshold (not exact == 0)

### Known traps
- **#11: Missing sheet prefix** — `=IF(N138>0,...)` vs `=IF(Model!N138>0,...)`
- Exact equality check instead of threshold for reconciliation

---

## Step 18: Full Integration Audit

NOT a build step — comprehensive final verification.

### Scope
1. **Driver cascade**: Change Inputs!F7 to 1, 2, 3 — verify ALL downstream changes
2. **Cross-sheet integrity**: Every green cell → verify source
3. **Historical verification**: Every blue cell → verify vs research file
4. **Formula consistency**: Same pattern across all forecast columns per row
5. **Formatting sweep**: Every cell font color, border, number format
6. **Corkscrew integrity**: All Opening = Prior Closing (WC, PP&E, Debt, Tax)
7. **Sign convention**: Consistent across all schedules
8. **Sensitivity spot-check**: 3 random cells manually computed
9. **Cover checks**: All 7 = PASS
10. **UFCF reconciliation**: Both methods tied across all forecast years
