# Step 18: Full Integration Audit

NOT a build step. Comprehensive end-to-end verification of the complete model. This is the final quality gate before delivery.

## Scope -- 10 Audit Passes

### Pass 1: Driver Cascade
- Change `Inputs!F7` to 1 (Best), verify ALL downstream values change
- Change to 3 (Worst), verify ALL downstream values change
- Change back to 2 (Base), verify original values restored
- Every forecast cell that depends on drivers should flex

### Pass 2: Cross-Sheet Integrity
- Every GREEN font cell: read the formula, verify it references the correct source sheet and row
- Check that sheet prefix is present (`Model!`, `Inputs!`) on every cross-sheet formula
- Verify on Outputs and Cover sheets especially (#11)

### Pass 3: Historical Verification
- Every BLUE font cell: compare value to research file
- Revenue, COGS, SGA, Other Opex, EBITDA, D&A, EBIT, Interest, Tax, NI
- AR, Inventory, AP, PP&E, Debt, Cash from BS
- No historical cell should contain a formula (must be hardcoded GAAP)
- Confirm historical IS was NOT overwritten by downstream schedules (#13)

### Pass 4: Formula Consistency
- For each forecast row: read formula in col J, K, L, M, N
- All 5 columns should follow identical pattern (only column letter changes)
- Flag any column that deviates from the pattern

### Pass 5: Formatting Sweep
- Blue font (#0000CC): hardcoded inputs only
- Black font (#000000): formulas within same sheet
- Green font (#006100): cross-sheet references
- Double borders on subtotals (EBITDA, NI, UFCF, Per Share, etc.)
- Number formats: currency/number with commas, percentages with 1 decimal

### Pass 6: Corkscrew Integrity
- Working Capital (AR, Inv, AP): Opening = Prior Closing
- PP&E (Existing, New, Summary): Opening = Prior Closing
- Debt: Opening = Prior Closing
- Tax Loss CF (Levered + Unlevered): Opening = Prior Closing
- First forecast Opening = last historical Closing for ALL corkscrews

### Pass 7: Sign Convention
- Costs: consistently negative (or positive — pick one)
- Depreciation: negative (reduces NBV and EBITDA)
- Interest: negative (expense)
- Tax: negative (expense)
- Capex: positive in PP&E (addition), negative in UFCF (cash outflow)
- Change in NWC: negative if WC increases (cash tied up)
- D&A add-back: positive in UFCF NI method

### Pass 8: Sensitivity Spot-Check
- Pick 3 random sensitivity cells
- Manually compute the expected value:
  - Re-discount each FCF at the cell's WACC
  - Compute TV at the cell's TGR/Multiple
  - Apply EV bridge
  - Divide by shares
- Compare to cell value. Must match within $0.01

### Pass 9: Cover Checks
- All 7 checks should show "PASS"
- If any show "FAIL": investigate and fix the root cause
- "FLAG" or "REVIEW" are acceptable if documented

### Pass 10: UFCF Reconciliation
- Method 1 = Method 2 across ALL forecast columns (not just the last one)
- Difference < $0.01 in every column
- If off: the difference often equals the deferred tax amount (#19 smoking gun)

## Audit Checklist (auditor MUST address EVERY item)

**Pass 1: Driver Cascade (full data flow trace)**
1. Set Inputs!F7 to 1 (Best Case); read 5+ downstream forecast cells and confirm ALL changed from Base values
2. Set Inputs!F7 to 3 (Worst Case); read the same cells and confirm ALL changed again (different from both Base and Best)
3. Set Inputs!F7 back to 2 (Base Case); confirm ALL cells restored to original values exactly
4. Verify at least one cell from each schedule (IS, BS, WC, PP&E, Tax, UFCF, DCF) flexed during toggle

**Pass 2: Cross-Sheet Integrity**
5. Every green-font cell (#006100) contains a formula with sheet prefix (`Model!`, `Inputs!`, `Cover!`) -- Trap #11
6. Read formulas on the Outputs sheet: every cross-sheet reference includes the sheet prefix
7. Read formulas on the Cover sheet: every cross-sheet reference includes the sheet prefix
8. Each green-font formula references the CORRECT source row (trace at least 10 green cells to their origin)

**Pass 3: Historical Verification**
9. Every blue-font cell (#0000CC) in the IS matches the corresponding value in the research CSV/file
10. Every blue-font cell in the BS matches the research file (AR, Inventory, AP, PP&E, Debt, Cash)
11. Historical IS cells contain hardcoded values (openpyxl: `cell.value` is a number, NOT a string starting with `=`) -- Trap #13
12. Historical BS cells contain hardcoded values (no formulas)

**Pass 4: Formula Consistency**
13. For each forecast row: read formulas in columns J, K, L, M, N; confirm identical pattern (only column letter changes)
14. Flag any column that deviates from the row's formula pattern (document the row and column)

**Pass 5: Formatting Sweep**
15. Blue font (#0000CC) used ONLY on hardcoded input cells; no formula cell has blue font
16. Black font (#000000) used on within-sheet formulas; no cross-sheet formula has black font
17. Green font (#006100) used on ALL cross-sheet references; no cross-sheet formula has non-green font
18. Double bottom borders present on ALL subtotal rows (EBITDA, Net Income, Total UFCF, Per Share, Tax Shield, Unlevered Total Tax)
19. Number formats: financial values use `#,##0` or `$#,##0`; percentages use `0.0%`; no raw unformatted decimals

**Pass 6: Corkscrew Integrity**
20. Working Capital (AR, Inventory, AP): every Opening = prior column Closing; first forecast Opening = last historical value
21. PP&E (Existing, New, Summary): every Opening = prior column Closing; first forecast Opening = last historical NBV
22. Debt: every Opening = prior column Closing; first forecast Opening = last historical debt balance
23. Tax Loss CF (Levered): every Opening = prior column Closing
24. Tax Loss CF (Unlevered): every Opening = prior column Closing
25. Verify at least one corkscrew chain end-to-end (Opening in col J through Closing in col N)

**Pass 7: Sensitivity Spot-Check**
26. Select 3 random sensitivity table cells (different WACC/TGR or WACC/Multiple combinations)
27. For each: manually compute PV of each discrete FCF at the cell's WACC
28. Compute TV at the cell's TGR (or Multiple), discount at the cell's WACC using end-of-year period
29. Apply EV bridge (Sum PV + PV TV - Debt + Cash) / Shares; compare to cell value; must match within $0.01

**Pass 8: Cover Checks + Reconciliation**
30. All 7 Cover sheet checks evaluate to "PASS" (read each cell value)
31. If any check shows "FAIL": document the root cause and fix it before proceeding
32. UFCF Reconciliation (Method 1 - Method 2) < $0.01 in EVERY forecast column (J through N), not just the last column
33. If reconciliation is off: check whether the difference equals the deferred tax amount -- Trap #19 smoking gun

**Pass 9: Sensitivity Tables + Outputs**
34. 8 sensitivity tables present on the Outputs sheet (4 for Perpetuity method x 2 axes, 4 for Multiple method x 2 axes)
35. Cumulative scenario comparison table present (Base / Best / Worst side by side)

**Pass 10: Final**
36. Model file saved as .xlsx with no unsaved formula changes
37. No orphaned references (`#REF!`, `#NAME?`, `#VALUE!`) anywhere in the workbook (search all cells)

## Known Traps (summary of ALL known mistakes)
- **#1: Depreciation / Tax Rate** -- Check Step 9 divisor
- **#2: Opening = Depreciation row** -- Check all corkscrews
- **#3: Single-year scenario scaling** -- Check Outputs scenario comparison
- **#4: Linear WACC scaling** -- Check sensitivity cell formulas
- **#5: NWC too low** -- Compare to historical ratios
- **#6: TV at mid-year** -- Check TV period in Steps 14-15
- **#7: Adjusted EBITDA** -- Compare EBITDA to GAAP IS
- **#8: Wrong PP&E** -- Compare to audited BS
- **#9: Fabricated peers** -- Check source comments
- **#10: Shares inconsistency** -- Trace all share refs to one cell
- **#11: Missing sheet prefix** -- Check Cover + Outputs formulas
- **#12: Hardcoded values in formulas** -- Check DCF bridge
- **#13: Historical IS overwritten** -- Check historical columns still blue
- **#14: Number not text for scenario** -- Check CHOOSE label
- **#15: Hardcoded peer stats** -- Check AVERAGE/MEDIAN formulas
- **#16: Built without reading reference template** -- Architecture check
- **#17: Numbers not traceable** -- Check cell comments
- **#18: Audit read script not Excel** -- Audit MUST open .xlsx with openpyxl
- **#19: Deferred tax missing** -- Check tax schedules have deferred section
- **#20: Half-year convention missing** -- Check new asset depreciation
- **#21: Inv/AP revenue-driven** -- Check WC closing formulas use COGS
