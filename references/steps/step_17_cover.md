# Step 17: Cover Sheet + Model Checks

Model metadata, company information, and 7 automated integrity checks with conditional formatting.

## Structure
```
--- Metadata ---
Row {title}:    Model Title (e.g., "{COMPANY} - DCF Valuation Model")
Row {ticker}:   Ticker & Exchange
Row {date}:     Model Date / Last Updated
Row {analyst}:  Analyst / Author
Row {blank}:    (blank)

--- Model Checks ---
Row {chk_hdr}:  "MODEL INTEGRITY CHECKS" header (bold, bottom border)
Row {blank}:    (blank)
Row {chk1}:     Check 1: Valuation Date Populated
Row {chk2}:     Check 2: Cash Flow Dates Populated
Row {chk3}:     Check 3: TGR < WACC
Row {chk4}:     Check 4: Debt Capacity
Row {chk5}:     Check 5: No Unused Levered Tax Losses
Row {chk6}:     Check 6: No Unused Unlevered Tax Losses
Row {chk7}:     Check 7: UFCF Reconciliation
```

## Key Formulas -- The 7 Checks

All check formulas MUST include sheet prefix (`Inputs!` or `Model!`).

### Check 1: Valuation Date Populated
```
=IF(Inputs!F{val_date}<>"", "PASS", "FAIL")
```

### Check 2: Cash Flow Dates Populated
```
=IF(Inputs!J{cf_date}<>"", "PASS", "FAIL")
```

### Check 3: TGR < WACC
```
=IF(Inputs!F{tgr} < Inputs!F{wacc}, "PASS", "FAIL")
```

### Check 4: Debt Capacity
```
=IF(Model!N{debt_close} >= 0, "PASS", "REVIEW")
```
Note: Returns "REVIEW" (not "FAIL") because negative debt may be intentional.

### Check 5: No Unused Levered Tax Losses
```
=IF(Model!N{lev_tl_close} <= 0, "PASS", "FLAG")
```
Note: Returns "FLAG" because remaining losses are not an error, but worth noting.

### Check 6: No Unused Unlevered Tax Losses
```
=IF(Model!N{ulev_tl_close} <= 0, "PASS", "FLAG")
```

### Check 7: UFCF Reconciliation
```
=IF(ABS(Model!N{recon}) < 0.01, "PASS", "FAIL")
```
Note: Uses ABS with threshold (not exact == 0) to handle floating-point rounding.

## Conditional Formatting

Apply to ALL check result cells:

| Result | Fill Color | Font Color |
|--------|-----------|------------|
| "PASS" | #C6EFCE (light green) | #006100 (dark green) |
| "FAIL" | #FFC7CE (light red) | #9C0006 (dark red) |
| "FLAG" / "REVIEW" | #FFEB9C (light yellow) | #9C6500 (dark yellow) |

Implementation:
```python
from openpyxl.formatting.rule import CellIsRule
from openpyxl.styles import PatternFill, Font

# Apply to each check cell
ws.conditional_formatting.add(cell_range,
    CellIsRule(operator='equal', formula=['"PASS"'],
              fill=PatternFill(start_color='C6EFCE', end_color='C6EFCE', fill_type='solid'),
              font=Font(color='006100')))
# ... similar for FAIL and FLAG/REVIEW
```

## Audit Checklist (auditor MUST address EVERY item)

1. Model Title row contains the company name (not blank, not a placeholder like "{COMPANY}")
2. Ticker & Exchange row is populated with the correct ticker symbol
3. Model Date row is populated (not blank)
4. Analyst / Author row is populated (not blank)
5. Check 1 formula: references `Inputs!` sheet for Valuation Date cell; verify sheet prefix present -- Trap #11
6. Check 1 result: returns "PASS" when Valuation Date is populated, "FAIL" when empty
7. Check 2 formula: references `Inputs!` sheet for Cash Flow Date cell; verify sheet prefix present -- Trap #11
8. Check 2 result: returns "PASS" when CF Date is populated, "FAIL" when empty
9. Check 3 formula: compares TGR vs WACC using `Inputs!` sheet prefix on BOTH cell references
10. Check 3 result: returns "PASS" when TGR < WACC, "FAIL" when TGR >= WACC
11. Check 4 formula: references `Model!` sheet for last forecast Debt Closing; verify sheet prefix present
12. Check 4 result: returns "REVIEW" (not "FAIL") when debt closing is negative
13. Check 5 formula: references `Model!` sheet for last forecast Levered Tax Loss Closing; verify sheet prefix
14. Check 5 result: returns "FLAG" (not "FAIL") when unused levered tax losses remain
15. Check 6 formula: references `Model!` sheet for last forecast Unlevered Tax Loss Closing; verify sheet prefix
16. Check 6 result: returns "FLAG" (not "FAIL") when unused unlevered tax losses remain
17. Check 7 formula: uses ABS() with threshold < 0.01 (NOT exact == 0); references `Model!` reconciliation row
18. Check 7 result: returns "PASS" when UFCF methods reconcile within $0.01
19. ALL 7 check cells contain IF formulas (openpyxl: `cell.value` starts with `=IF`); NO hardcoded "PASS"/"FAIL" strings
20. Every formula across all 7 checks uses sheet prefix (`Inputs!` or `Model!`) -- Trap #11; read each formula string and confirm
21. Conditional formatting rule exists for "PASS": fill #C6EFCE, font #006100 (openpyxl: inspect `ws.conditional_formatting`)
22. Conditional formatting rule exists for "FAIL": fill #FFC7CE, font #9C0006
23. Conditional formatting rule exists for "FLAG" and "REVIEW": fill #FFEB9C, font #9C6500
24. Conditional formatting range covers ALL 7 check result cells (not a subset)
25. For a correctly built model, all 7 checks evaluate to "PASS" (read computed values if possible)
26. Row constants (not literal row numbers) used in all formula references; verify no magic numbers in formula strings

## Known Traps
- **#11: Missing sheet prefix** -- `=IF(N138>0,...)` fails when Cover is active sheet. Must be `=IF(Model!N138>0,...)`.
- Exact equality check for reconciliation instead of ABS < threshold (floating-point rounding can produce tiny non-zero values)
- Hardcoding "PASS" instead of using a formula (defeats the purpose of automated checks)
- Using literal row numbers instead of row constants (breaks if skeleton layout changes)
