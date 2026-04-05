# Step 12: Unlevered Tax + Tax Shield

Tax from EBIT (no interest deduction) + Tax Shield calculation. Same deferred tax structure as Step 11 but based on EBIT.

## Structure
```
Row {hdr}: "TAX SCHEDULE — UNLEVERED" header (bold, bottom border)

--- Current Tax ---
Row {ebit}:           EBIT (green -> IS)
Row {utl_open}:       Unlevered Tax Loss Opening
Row {utl_used}:       Tax Loss Used = MIN(Opening, MAX(0, EBIT))
Row {utaxable}:       Unlevered Taxable = MAX(0, EBIT) - Loss Used
Row {ucurrent_tax}:   Unlevered Current Tax = Taxable * Rate (negative)
Row {utl_close}:      Unlevered Tax Loss Closing

--- Deferred Tax ---
Row {uacct_depr}:     Accounting Depreciation (green -> same as levered)
Row {utax_depr}:      Tax/CCA Depreciation (green -> same as levered)
Row {utiming}:        Timing Difference
Row {udeferred}:      Unlevered Deferred Tax

--- Total ---
Row {utotal_tax}:     Unlevered Total Tax = Current + Deferred (double border)

--- Tax Shield ---
Row {shield}:         Tax Shield = Unlevered Total Tax - Levered Total Tax (positive)
```

## Tax Shield Direction (CRITICAL)

```
Tax Shield = Unlevered Tax - Levered Tax
```

This is **POSITIVE** because:
- Unlevered firm pays MORE tax (no interest deduction from EBT)
- Levered firm pays LESS tax (interest reduces EBT, reducing tax)
- The difference = tax savings from having debt

**DO NOT write `Levered - Unlevered`** — that gives a negative number and inverts the shield.

Equivalent: `Tax Shield ≈ Interest Expense * Tax Rate` (only exact when no tax losses)

## Audit Checklist (auditor MUST address EVERY item)

1. EBIT source cell references the Income Statement EBIT row (NOT EBT -- unlevered means no interest deduction); verify green font + sheet prefix on the formula
2. Statutory tax rate cell references the same Inputs cell used in levered tax (Step 11); confirm identical rate in both schedules
3. Unlevered Tax Loss Opening in first forecast column = prior column Closing (corkscrew link intact)
4. Unlevered Tax Loss Used = MIN(Opening, MAX(0, EBIT)) -- verify the formula literally, not just the result
5. Unlevered Taxable Income = MAX(0, EBIT) - Loss Used -- no negative taxable income possible
6. Unlevered Current Tax = Taxable * Statutory Rate; result is NEGATIVE (expense convention)
7. Unlevered Tax Loss Closing = Opening - Used + any new losses; corkscrew balances in every forecast column
8. Accounting Depreciation row references the SAME source cell as the levered schedule (green font, sheet prefix)
9. Tax/CCA Depreciation row references the SAME source cell as the levered schedule (green font, sheet prefix)
10. Timing Difference = Accounting Depreciation - Tax Depreciation (same formula structure as Step 11)
11. Unlevered Deferred Tax = Timing Difference * Statutory Rate (same methodology as levered)
12. Unlevered Total Tax = Current Tax + Deferred Tax; verify the SUM formula explicitly
13. Double bottom border applied to Unlevered Total Tax row (openpyxl: `cell.border.bottom.style == 'double'`)
14. Tax Shield = Unlevered Total Tax - Levered Total Tax (NOT Levered - Unlevered); result is POSITIVE
15. Tax Shield row has double bottom border (openpyxl: `cell.border.bottom.style == 'double'`)
16. Tax Shield sanity check: value approximately equals Interest Expense * Tax Rate (exact only when no tax losses active)
17. All green-font cells on this schedule have font color #006100 (openpyxl: `cell.font.color.rgb` ends with `006100`)
18. All green-font cells include sheet prefix in their formulas (`Model!` or `Inputs!`)
19. Unlevered Total Tax value flows correctly to the UFCF EBITDA method schedule (trace the reference)
20. Formula pattern is consistent across all 5 forecast columns (J through N) for every row

## Known Traps
- **Tax Shield direction reversed** (Tax Shield direction error — see step spec line 33) — `Levered - Unlevered` = wrong sign
- Using different rates for levered vs unlevered
- Forgetting deferred tax on unlevered side (same timing differences apply)
