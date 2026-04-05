# Step 10: Debt Schedule (Model Sheet)

Debt corkscrew plus interest calculation. CRITICAL: formulas ONLY in forecast columns — must not overwrite historical IS values.

## Structure
```
Row {hdr}:      "DEBT SCHEDULE" header (bold, bottom border)
Row {blank}:    (blank)
Row {open}:     Debt Opening
Row {draws}:    Draws / New Proceeds
Row {repay}:    Repayments
Note: Repayment assumptions should come from Inputs (mandatory amortization from credit facility terms). Do NOT default to 0 repayment without confirming facility terms.
Row {close}:    Debt Closing = Opening + Draws - Repayments (double border)
Row {blank}:    (blank)
Row {rate}:     Interest Rate (green -> Inputs)
Row {interest}: Interest Expense = Opening * Rate (negative, links to IS)
```

## Key Formulas

### Historical (columns F-I)
- Opening, Draws, Repayments, Closing: hardcoded from BS / CF statement, BLUE font
- Interest: hardcoded from IS, BLUE font (GAAP interest expense)
- Rate: implied = Interest / Opening (or hardcoded)

### Forecast (columns J-N ONLY)
- Opening: `={PREV}${DEBT_CLOSE}` (prior year Debt Closing)
- Draws: blue input or 0 (assumptions about future borrowing)
- Repayments: blue input or scheduled (from facility terms)
- Closing: `={COL}{DEBT_OPEN}+{COL}{DEBT_DRAWS}-{COL}{DEBT_REPAY}`
- Rate: `=Inputs!$F${DEBT_RATE}` (green, from Other Inputs)
- Interest: `=-((({COL}{DEBT_OPEN}+{COL}{DEBT_CLOSE})/2)*{COL}{DEBT_RATE})` (negative = expense)
  Interest = Average Balance * Rate = ((Opening + Closing) / 2) * Rate
  Using Opening-only balance overstates interest when debt declines. Average balance is standard practice.

### Linking to IS
- Interest expense row in IS (Step 7) should reference this schedule's interest row
- **ONLY forecast columns** — historical IS interest must remain hardcoded GAAP

## Audit Checklist (openpyxl-verifiable)
1. Historical debt from CSV (blue font, F-I only) — verify `ws.cell(DEBT_CLOSE, col).value` matches CSV for columns F-I; `cell.font.color.rgb == '000000CC'`; same for Opening, Draws, Repayments, Interest
2. Forecast: Opening = Prior Closing — verify `ws.cell(DEBT_OPEN, col).value` references prior column's `DEBT_CLOSE` row for columns J-N; first forecast (col J) references col I
3. Repayment from Inputs (negative) — verify `ws.cell(DEBT_REPAY, col).value` for forecast columns references Inputs sheet or is a negative hardcoded assumption; value should be <= 0
4. Closing = Opening + Draws + Repayments — verify `ws.cell(DEBT_CLOSE, col).value` contains formula referencing `DEBT_OPEN`, `DEBT_DRAWS`, and `DEBT_REPAY` rows for forecast columns J-N
5. Interest = (Opening + Closing) / 2 * Rate (average balance, Trap #25) — verify `ws.cell(DEBT_INTEREST, col).value` contains formula with `(Opening+Closing)/2` pattern, NOT Opening-only; must reference both `DEBT_OPEN` and `DEBT_CLOSE`
6. Historical interest from CSV — verify `ws.cell(DEBT_INTEREST, col).value` for columns F-I is a hardcoded number (not a formula) matching CSV; `cell.font.color.rgb == '000000CC'`
7. FCST_COLS only for forecast formulas (Trap #13) — verify `ws.cell(DEBT_INTEREST, col).value` for historical columns F-I is numeric (isinstance(value, (int, float))), NOT a formula string starting with `=`; historical IS interest must remain untouched
8. Double border on Closing — verify `ws.cell(DEBT_CLOSE, col).border.bottom.style == 'double'` for all data columns F-N
9. Source comments reference CSV — verify `ws.cell(row, col).comment is not None` for historical cells (F-I) in Opening, Closing, and Interest rows; comment text contains source reference

## Known Traps
- **#13: Historical IS overwritten** — The debt schedule formula loop MUST use `FCST_COLS` only. If it writes to ALL columns (F-N), it overwrites historical GAAP interest expense with formula values. After building this step, RE-CHECK that historical IS columns F-I still contain blue hardcoded values.
- Interest on Closing balance instead of Opening (overstates interest in years with net draws)
- **#25: Interest on opening balance** — should use average of opening and closing. Opening-only overstates interest when debt declines from repayments.
- Mixing up rate source (using WACC instead of debt rate from facility terms)
- Forgetting that Draws increase debt and Repayments decrease it (sign handling)
