# Step 15: DCF -- EBITDA Multiple Method

Same discounting framework as Step 14 but terminal value uses exit EBITDA multiple instead of Gordon Growth.

## Structure
```
Row {hdr}:      "DCF -- EBITDA MULTIPLE METHOD" header (bold, bottom border)
Row {blank}:    (blank)

--- Discounting (identical to Step 14) ---
Row {dates}:    Cash Flow Dates (green -> Inputs)
Row {periods}:  Discount Periods (mid-year, same as Step 14)
Row {ufcf}:     UFCF (green -> UFCF Method 1)
Row {df}:       Discount Factor = 1/(1+WACC)^period
Row {pv}:       PV of FCF = UFCF * DF

--- Terminal Value (EBITDA Multiple) ---
Row {tv_lbl}:   "Terminal Value"
Row {last_ebitda}: Last Forecast EBITDA (green -> IS EBITDA, last forecast col)
Row {exit_mult}:   Exit EBITDA Multiple (green -> Inputs)
Row {tv}:       TV = Last EBITDA * Exit Multiple
Row {tv_period}: TV Discount Period (END-of-year integer, same as Step 14)
Row {pv_tv}:    PV of TV = TV / (1+WACC)^TV_Period

--- Enterprise Value Bridge (identical structure to Step 14) ---
Row {ev_lbl}:   "Enterprise Value Bridge"
Row {sum_pv}:   Sum of PV of FCFs
Row {pv_tv2}:   PV of Terminal Value
Row {ev}:       Enterprise Value = Sum PV + PV TV
Row {debt}:     Less: Debt (green -> Inputs, negative)
Row {cash}:     Plus: Cash (green -> Inputs, positive)
Row {equity}:   Equity Value
Row {shares}:   Shares Outstanding (green -> Inputs)
Row {price}:    Implied Share Price (double border)
```

## Key Formulas

### Terminal Value (key difference from Step 14)
```
TV = Last EBITDA * Exit Multiple
={LAST_COL}${IS_EBITDA} * Inputs!$F${EXIT_MULT}
```
- Uses EBITDA (not FCF) times the exit multiple
- EBITDA from the IS, last forecast column
- Exit multiple from Inputs

### PV of FCFs
- MUST be identical values to Step 14 (same UFCF, same periods, same WACC)
- Can reference Step 14's PV rows directly OR recompute (if recomputing, must match exactly)

### EV Bridge
- Same structure as Step 14
- Different EV because different TV
- Same Debt, Cash, Shares references

## Audit Checklist (auditor MUST address EVERY item)

1. UFCF row references the same UFCF source row as Step 14 (green font, sheet prefix); values must be identical
2. Cash Flow Date row references Inputs sheet dates (green font, sheet prefix `Inputs!`)
3. Discount periods use mid-year convention for discrete FCFs (e.g., 0.5, 1.5, 2.5...) -- identical to Step 14
4. Discount Factor formula = 1/(1+WACC)^period; WACC references Inputs with sheet prefix
5. PV of FCF = UFCF * Discount Factor; values match Step 14 PV row exactly (cell-by-cell comparison)
6. Terminal Value = Last Forecast EBITDA * Exit Multiple -- verify TV formula is literally `={last_col}${ebitda_row} * Inputs!$F${exit_mult_row}` (NOT FCF * Multiple -- Trap)
7. EBITDA source is the Income Statement EBITDA row in the last forecast column (green font, sheet prefix `Model!`)
8. Exit Multiple references the Inputs sheet cell (green font, sheet prefix `Inputs!`)
9. TV Discount Period = END-of-year integer (e.g., 5, not 4.5) -- Trap #6; verify it is NOT mid-year
10. PV of TV = TV / (1+WACC)^TV_Period; confirm the TV period used is the end-of-year integer
11. Sum of PV of FCFs formula covers exactly the discrete forecast PV cells (no more, no fewer)
12. Enterprise Value = Sum of PV of FCFs + PV of Terminal Value
13. Debt cell references Inputs sheet (green font, sheet prefix `Inputs!`); value is negative; NOT hardcoded -- Trap #12
14. Cash cell references Inputs sheet (green font, sheet prefix `Inputs!`); value is positive; NOT hardcoded -- Trap #12
15. Equity Value = EV - Debt + Cash (or EV + Debt + Cash if Debt is already negative)
16. Shares Outstanding references the SAME Inputs cell as Step 14 (green font, sheet prefix) -- Trap #10
17. Implied Share Price = Equity Value / Shares Outstanding
18. Double bottom border on Per Share row (openpyxl: `cell.border.bottom.style == 'double'`)
19. Per Share value is DIFFERENT from Step 14 (since TV methodology differs); flag if identical
20. All green-font cells have font color #006100 (openpyxl: `cell.font.color.rgb` ends with `006100`)
21. All green-font formulas include sheet prefix (`Model!`, `Inputs!`) -- no bare cell references to other sheets
22. Number formats: currency cells use `#,##0` or `$#,##0`; percentages use `0.0%`

## Known Traps
- **Using FCF instead of EBITDA for exit multiple** -- Exit multiples are applied to EBITDA, not free cash flow. Using FCF would double-count capex/NWC deductions.
- **#6: TV at mid-year** -- Same trap as Step 14. TV period must be end-of-year integer.
- Inconsistent discount periods vs Step 14 (must be identical)
- **#12: Cash/Debt hardcoded** -- Same trap as Step 14.
- **#10: Shares inconsistency** -- Same source cell as Step 14.
