# Known Mistakes — AutoDCF Audit Checklist

18 catalogued bugs from 8+ adversarial audit rounds. Organized by severity. Each entry includes detection method for the audit agent.

---

## CATASTROPHIC (wrong valuation, silent failure)

### #1: Depreciation / Tax Rate instead of / Useful Life
**Bug**: Depreciation formula divides Opening NBV by the tax rate row (e.g., 26.5%) instead of useful life row (e.g., 10 years). These rows are often adjacent in Inputs.
**Impact**: Depreciation is ~37x too large (dividing by 0.265 vs 10). Cascades to PP&E, D&A, EBIT, EBITDA, Tax, UFCF, and final valuation.
**Detection**: Read the depreciation formula. Extract the Inputs row referenced as divisor. Verify it matches USEFUL_LIFE_ROW constant, not TAX_RATE_ROW.
**Steps affected**: 9 (Depreciation & PP&E)

### #2: Opening NBV = Depreciation Row instead of Closing Row
**Bug**: Opening PP&E formula references the depreciation row (`=I{depr}`) instead of closing NBV (`=I{close}`). Caused by arithmetic offset `r+1` when depreciation sits between opening and closing.
**Impact**: PP&E corkscrew is completely broken. All downstream values wrong.
**Detection**: Read the Opening formula. Verify the row number matches PPE_CLOSE_ROW, not DEPR_ROW or any other row.
**Steps affected**: 9

---

## STRUCTURAL (wrong methodology)

### #3: Single-Year Scenario Scaling instead of Cumulative
**Bug**: Bear/Bull outputs computed as `FCF * (1+bear_g) / (1+base_g)` per year. Should be cumulative product of all years' differentials through year t.
**Impact**: Scenario comparison is mathematically wrong — underestimates divergence in later years.
**Detection**: Check the scenario comparison formulas in Outputs. They should contain PRODUCT or nested multiplication across all prior years, not a single-year ratio.
**Steps affected**: 16 (Outputs Dashboard)

### #4: Sensitivity Tables Use Linear WACC Scaling
**Bug**: Sensitivity cell = `PV_FCFs * (WACC_base / WACC_new)`. This is a linear approximation that's wrong — DCF discounting is exponential.
**Impact**: Sensitivity values are meaningfully wrong, especially at edges of the table.
**Detection**: Read a sensitivity cell formula. It must contain per-year discounting terms `FCF/(1+WACC)^period` for each forecast year. If it contains a simple ratio or multiplication by base value, it's linear scaling.
**Steps affected**: 16

### #5: NWC Assumption Unrealistically Low
**Bug**: NWC as % of revenue assumed at ~16% when actual historical ratio was ~22%. Creates ~$120M phantom FCF.
**Impact**: UFCF inflated, valuation too high.
**Detection**: Compute NWC/Revenue from historical data. Compare to the assumption in Inputs. Flag if deviation > 3 percentage points.
**Steps affected**: 2 (Drivers), 8 (Working Capital)

### #6: Terminal Value Discounted at Mid-Year
**Bug**: TV discounted at period 4.5 (mid-year) instead of 5.0 (end-of-year). TV represents a perpetuity starting at the END of the last forecast year.
**Impact**: TV is discounted too little, inflating EV.
**Detection**: Read the TV discount period cell. For 5 forecast years, it should be exactly 5.0 (or equivalent integer), not 4.5 or fractional.
**Steps affected**: 14, 15 (DCF Perpetuity/Multiple)

---

## DATA (wrong inputs)

### #7: "Adjusted" EBITDA instead of GAAP
**Bug**: Used management's "adjusted" EBITDA (which excludes restructuring, impairments, etc.) instead of GAAP EBITDA computed from the actual income statement.
**Impact**: EBITDA overstated, valuation inflated.
**Detection**: Compare historical EBITDA in the model to GAAP IS figures (Revenue - COGS - SGA - Other Opex). If it doesn't tie, it may be "adjusted."
**Steps affected**: 7 (Income Statement), all downstream

### #8: Wrong PP&E Value (Estimate vs Actual)
**Bug**: PP&E opening balance was $85M (estimated) when actual audited balance was $44.8M.
**Impact**: Depreciation wrong, NBV wrong, capex ratios wrong.
**Detection**: Compare the PP&E opening balance in Inputs to the research file. Must match exactly.
**Steps affected**: 4 (Other Inputs), 9 (PP&E)

### #9: Fabricated Peer WACC Data
**Bug**: Peer company Market Cap, Debt, Cash, Beta values were estimated/guessed instead of sourced from actual market data.
**Impact**: WACC is unreliable, entire valuation questionable.
**Detection**: Check that each peer data cell has a source comment. Verify source is a real market data provider (not "estimated" or "assumed").
**Steps affected**: 3 (WACC Section)

### #10: Shares Outstanding Inconsistency
**Bug**: Different share counts used in different parts of the model (diluted vs basic, or different dates).
**Impact**: Per share values inconsistent.
**Detection**: Find all cells referencing shares outstanding. Verify they all ultimately trace to one source cell in Inputs.
**Steps affected**: 4, 14, 15, 16

---

## LINKAGE (broken references)

### #11: Cross-Sheet Formula Missing Sheet Prefix
**Bug**: Cover check formula `=IF(N138>0,...)` instead of `=IF(Model!N138>0,...)`. Works if Model is active sheet, fails otherwise.
**Impact**: Cover checks show wrong results or #REF!
**Detection**: For every formula on Cover and Outputs sheets, verify that cell references to other sheets include the sheet name prefix.
**Steps affected**: 17 (Cover), 16 (Outputs)

### #12: Hardcoded Values in Formula Cells
**Bug**: Cash ($8.3M), Debt balance, COGS%, SGA% hardcoded directly in calculation cells instead of referencing an input cell.
**Impact**: Model doesn't flex when assumptions change. Hidden assumptions.
**Detection**: In forecast formula cells, check that no literal numbers appear (except 0, 1, -1 for sign). Every number should be a cell reference.
**Steps affected**: 14, 15 (DCF bridge), various

### #13: Historical IS Overwritten by Downstream Schedule
**Bug**: Debt Schedule interest formula loop wrote formulas to ALL columns (F-N) including historical (F-I), overwriting GAAP interest expense.
**Impact**: Historical IS no longer matches actual filings.
**Detection**: After building each schedule, re-check historical IS columns. All should still be hardcoded GAAP values (blue font), not formulas.
**Steps affected**: 10 (Debt), any step that writes to IS-adjacent rows

---

## FORMAT (cosmetic but important)

### #14: Dashboard Shows Number Instead of Text for Scenario
**Bug**: Active scenario displayed as "2" instead of "Base Case". Need CHOOSE formula to convert.
**Impact**: Unprofessional, confusing for readers.
**Detection**: Check the scenario label cell in Outputs. Should contain a CHOOSE formula returning text strings.
**Steps affected**: 16 (Outputs)

### #15: Peer Statistics Hardcoded in Python
**Bug**: Average and Median unlevered beta computed in Python and written as values, instead of using Excel AVERAGE/MEDIAN formulas.
**Impact**: Statistics don't update if peer data changes. Not a live model.
**Detection**: Check the Average/Median cells. They should start with `=AVERAGE(` or `=MEDIAN(`, not be plain numbers.
**Steps affected**: 3 (WACC)

---

## PROCESS (methodology errors)

### #16: Built Without Reading CFI Template
**Bug**: Started coding without first inspecting the CFI reference template structure.
**Impact**: Architecture doesn't match CFI standard, requires complete rebuild.
**Prevention**: Always read the CFI template with openpyxl before writing any code.
**Steps affected**: All (architecture)

### #17: Numbers Not Traceable to Source
**Bug**: Hardcoded values with no comments indicating where they came from.
**Impact**: Cannot verify accuracy, cannot update when new filings available.
**Detection**: For every blue-font cell, check `cell.comment is not None`. Missing comments = fail.
**Steps affected**: All steps with hardcoded values

### #18: Audit Read Script Not Excel
**Bug**: Audit agent read the Python build script to verify formulas instead of reading the actual .xlsx output with openpyxl.
**Impact**: Script may not produce what you'd expect. Only the actual Excel file is truth.
**Prevention**: Audit agent MUST open .xlsx with openpyxl. NEVER trust the script as proxy.
**Steps affected**: All (audit methodology)

### #19: Deferred Tax Omitted — UFCF Reconciliation Fails
**Bug**: Tax schedules only compute current tax, omitting deferred tax from accounting vs tax depreciation timing differences. The NI method UFCF requires a Deferred Tax add-back. Without it, NI method != EBITDA method.
**Impact**: UFCF reconciliation check (which MUST = 0) will ALWAYS fail. The model's core integrity check is broken.
**Detection**: Check if tax schedules have Accounting Depreciation, Tax/CCA Depreciation, Timing Difference, and Deferred Tax rows. If only Current Tax exists, deferred tax is missing.
**Steps affected**: 11 (Levered Tax), 12 (Unlevered Tax), 13 (UFCF)

### #20: New Asset Depreciation Missing Half-Year Convention
**Bug**: New capex in year 1 gets a full year of depreciation instead of half-year. Standard accounting convention is that assets acquired mid-year get 50% depreciation in the acquisition year.
**Impact**: First-year depreciation overstated by ~50% of capex depreciation. Cascades to NBV, D&A, EBIT, Tax, UFCF.
**Detection**: Check new-asset depreciation formula. It should contain `* 0.5` multiplier on current-year capex: `-Capex * Rate * 0.5`. If it's `-Capex * Rate` (no 0.5), half-year convention is missing.
**Steps affected**: 9 (Depreciation & PP&E)

### #21: Inventory/AP Driven by Revenue Instead of COGS
**Bug**: Working capital forecast uses Revenue * NWC% for ALL components. Inventory and AP should be driven by COGS (or Cost of Revenue), not Revenue. Only AR is revenue-driven.
**Impact**: When COGS margin shifts, WC forecast diverges from reality. Can create material FCF errors.
**Detection**: Check WC closing formulas. Inventory Closing should reference COGS row (not Revenue). AP Closing should reference COGS row. AR Closing should reference Revenue.
**Steps affected**: 8 (Working Capital), 2 (Drivers)

### #22: UFCF Tax Component Mismatch (Current vs Total)
**Bug**: EBITDA method uses Unlevered Total Tax while NI method uses Levered Deferred Tax + Full Tax Shield. The deferred component appears in both methods with incompatible signs, preventing algebraic cancellation.
**Impact**: UFCF reconciliation fails. Agent wastes 3-4 iterations debugging a formula that CANNOT work because components are from different approaches.
**Detection**: If reconciliation != 0, check if EBITDA method uses Current or Total unlevered tax. Then verify NI method uses CORRESPONDING components: Current tax => Decomposed (Deferred + Current Shield). Total tax => Simplified (Zero Deferred + Full Shield).
**Steps affected**: 13 (UFCF), 12 (Unlevered Tax), 11 (Levered Tax)

### #23: Audit Batching Defeats Per-Step Quality Control
**Bug**: Agent codes 4-8+ steps between audits, or backgrounds audit agents and continues building. By the time the integration audit catches a bug at Step 18, 10+ steps are built on a broken foundation.
**Impact**: Single formula error in Step 9 cascades through Steps 10-17. Catching it at Step 18 means debugging 10 steps instead of 1.
**Detection**: Check BUILD_STATE.md — if multiple steps show "passed, 1 iter" simultaneously or steps 7-17 have no individual audit record, audits were batched.
**Steps affected**: All (process)

### #24: PP&E Opening Includes ROU Assets (IFRS 16)
**Bug**: Total "Property, plant and equipment" from balance sheet ($235M) used for PP&E depreciation schedule, but this includes Right-of-Use assets (~$136M) which are lease obligations. Depreciable PP&E is only ~$85M.
**Impact**: Depreciation overstated by 2-3x. D&A, EBIT, NI, deferred tax, UFCF all wrong. Terminal value overstated via inflated EBITDA.
**Detection**: Compare PP&E input to the filing's note disclosure. If PP&E includes "right-of-use" or "lease" assets, it's inflated.
**Steps affected**: 0 (Research), 4 (Other Inputs), 9 (Depreciation)

### #25: Interest on Opening Balance Not Average
**Bug**: Debt interest = Opening Balance * Rate. When debt declines from repayments, this overstates interest (uses beginning-of-year debt when end-of-year is lower). Standard practice uses average balance.
**Impact**: Interest overstated, EBT/NI understated, UFCF slightly affected.
**Detection**: Check debt interest formula — should be `((Open+Close)/2)*Rate`, not `Open*Rate`.
**Steps affected**: 10 (Debt Schedule)
