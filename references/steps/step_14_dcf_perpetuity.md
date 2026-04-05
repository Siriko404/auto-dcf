# Step 14: DCF -- Perpetuity Growth Method

Gordon Growth terminal value, mid-year FCF discounting, end-of-year TV discounting, and EV-to-equity bridge.

## Structure
```
Row {hdr}:      "DCF -- PERPETUITY GROWTH METHOD" header (bold, bottom border)
Row {blank}:    (blank)

--- Discounting ---
Row {dates}:    Cash Flow Dates (green -> Inputs, July 1 of each forecast year)
Row {periods}:  Discount Periods = (CF Date - Valuation Date) / 365.25
Row {ufcf}:     UFCF (green -> UFCF Method 1, Step 13)
Row {df}:       Discount Factor = 1 / (1 + WACC) ^ period
Row {pv}:       PV of FCF = UFCF * Discount Factor

--- Terminal Value ---
Row {tv_lbl}:   "Terminal Value"
Row {last_fcf}: Last Forecast UFCF (green -> last column UFCF)
Row {tgr}:      Terminal Growth Rate (green -> Inputs)
Row {wacc}:     WACC (green -> Inputs)
Row {tv}:       TV = Last FCF * (1 + TGR) / (WACC - TGR)
Row {tv_period}: TV Discount Period (END-of-year integer, NOT mid-year)
Row {pv_tv}:    PV of TV = TV / (1 + WACC) ^ TV_Period

--- Enterprise Value Bridge ---
Row {ev_lbl}:   "Enterprise Value Bridge"
Row {sum_pv}:   Sum of PV of FCFs
Row {pv_tv2}:   PV of Terminal Value (green -> above)
Row {ev}:       Enterprise Value = Sum PV + PV TV
Row {debt}:     Less: Debt (green -> Inputs or Debt Closing, negative)
Row {cash}:     Plus: Cash (green -> Inputs, positive)
Row {equity}:   Equity Value = EV - Debt + Cash
Row {shares}:   Shares Outstanding (green -> Inputs)
Row {price}:    Implied Share Price = Equity / Shares (double border)
```

## Key Formulas

### Mid-Year Cash Flow Date Convention
Cash flow dates = **July 1 of each forecast year** (e.g., 2026-07-01, 2027-07-01, ...). This represents the mid-point of each annual cash flow, producing fractional discount periods (approximately 0.5, 1.5, 2.5, 3.5, 4.5 for a model valued at year-end).

### Discount Periods
```
=(Inputs!{COL}${CF_DATE} - Inputs!$F${VAL_DATE}) / 365.25
```
- Produces fractional mid-year periods (e.g., 0.5, 1.5, 2.5, 3.5, 4.5)
- Uses 365.25 (accounts for leap years)

### Discount Factor
```
=1/(1+Inputs!$F${WACC})^{COL}{PERIOD}
```

### PV of FCF
```
={COL}{UFCF}*{COL}{DF}
```

### Terminal Value (Gordon Growth)
```
={LAST_COL}{UFCF}*(1+Inputs!$F${TGR})/(Inputs!$F${WACC}-Inputs!$F${TGR})
```

### TV Discount Period -- CRITICAL
```
TV Period = integer N (e.g., 5.0 for 5 forecast years)
```
- This is the END-of-year period, NOT mid-year
- TV represents a perpetuity starting at the END of the last forecast year
- For 5 forecast years with mid-year FCF dates: FCF periods are 0.5,1.5,2.5,3.5,4.5 but TV period is 5.0

### PV of Terminal Value
```
={TV}/(1+Inputs!$F${WACC})^{TV_PERIOD}
```

### EV Bridge
```
EV = SUM(PV of FCFs) + PV of TV
Equity = EV - Debt + Cash
Per Share = Equity / Shares
```
- Debt and Cash MUST reference INPUT cells (not hardcoded values)

## Audit Checklist (openpyxl-verifiable)
1. Discount periods computed from (CF Date - Val Date) / 365.25 — verify `ws.cell(PERIOD_ROW, col).value` contains formula `=(Inputs!{COL}${CF_DATE}-Inputs!$F${VAL_DATE})/365.25` for each forecast column
2. UFCF references Method 1 totals — verify `ws.cell(UFCF_ROW, col).value` contains formula referencing the UFCF Method 1 total row (e.g., `={COL}${UFCF_M1_TOTAL}`); font color is green (`cell.font.color.rgb == '00006100'`)
3. TV = Last FCF * (1+g) / (WACC - g) [Gordon Growth] — verify `ws.cell(TV_ROW, col).value` contains `*(1+` and `/(` pattern matching Gordon Growth; must reference TGR and WACC from Inputs with `Inputs!` prefix
4. TV discount period = END-OF-YEAR integer (e.g., 5.0), NOT mid-year (Trap #6) — verify `ws.cell(TV_PERIOD_ROW, col).value` is an integer (e.g., 5) or formula that evaluates to an integer, NOT a mid-year fractional value like 4.5
5. EV = Sum PV FCFs + PV TV — verify `ws.cell(EV_ROW, col).value` contains `SUM` of PV FCF range plus PV of TV reference
6. Equity = EV - Debt + Cash (from Inputs) — verify `ws.cell(EQUITY_ROW, col).value` references `EV_ROW`, debt cell, and cash cell; debt and cash must contain `Inputs!` prefix (NOT hardcoded numbers)
7. Per Share = Equity / Shares * 1000 — verify `ws.cell(PRICE_ROW, col).value` contains formula dividing Equity by Shares with appropriate unit conversion; Shares reference must contain `Inputs!` prefix
8. All Inputs references use sheet prefix and green font — verify every cell containing `Inputs!` in its formula has `cell.font.color.rgb == '00006100'`; grep all formulas in the DCF section for bare row references that should have `Inputs!` prefix
9. Double border on Per Share row — verify `ws.cell(PRICE_ROW, col).border.bottom.style == 'double'` for all data columns
10. Shares consistent across both DCF methods — verify the Shares row reference in Perpetuity method (`ws.cell(SHARES_ROW_PERP, col).value`) points to the same Inputs cell as the Multiple method's Shares row; both must use identical `Inputs!$F${SHARES}` reference

## Known Traps
- **#6: TV discounted at mid-year (4.5) instead of end-of-year (5.0)** -- TV is a perpetuity starting at year-end. Using mid-year discounting understates the discount and inflates EV.
- **#12: Cash/Debt hardcoded in EV bridge** -- Must reference input cells so the model flexes.
- **#10: Shares inconsistency** -- Must reference the same Inputs source cell as all other share references.
- **#11: Missing sheet prefix** -- Every cross-sheet reference must include `Inputs!` or `Model!`.
- Forgetting `* (1+TGR)` in Gordon Growth numerator (using last FCF directly instead of grown FCF)
