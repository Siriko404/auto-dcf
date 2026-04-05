# Step 7: Income Statement (Model Sheet)

Full IS from Revenue through Net Income. Pulls from Revenue Schedule, Cost Schedule, and forward-references Depreciation, Debt, and Tax Schedules.

## Structure
```
Row {hdr}:       "INCOME STATEMENT" header (bold, bottom border)
Row {blank}:     (blank)
Row {rev}:       Revenue (green -> Revenue Schedule)
Row {cogs}:      COGS (green -> Cost Schedule)
Row {gross}:     Gross Profit = Revenue + COGS (if costs negative) or Revenue - COGS
Row {sga}:       SGA (green -> Cost Schedule)
Row {other}:     Other Opex (green -> Cost Schedule)
Row {ebitda}:    EBITDA (double border)
Row {ebitda_chk}: EBITDA Check = Rev + COGS + SGA + Other (verify ties to EBITDA)
Row {da}:        D&A (green -> Depreciation Schedule, Step 9) [FORWARD REF]
Row {ebit}:      EBIT = EBITDA + D&A (D&A is negative)
Row {interest}:  Interest Expense (green -> Debt Schedule, Step 10) [FORWARD REF]
Row {ebt}:       EBT = EBIT + Interest (Interest is negative)
Row {tax}:       Tax Expense (green -> Levered Tax Schedule, Step 11) [FORWARD REF]
Row {ni}:        Net Income = EBT + Tax (Tax is negative) (double border)
```

## Key Formulas

### Historical (columns F-I)
- ALL rows: hardcoded GAAP values, BLUE font, source comments
- EBITDA Check: `={COL}{REV}+{COL}{COGS}+{COL}{SGA}+{COL}{OTHER}` (should tie to EBITDA even for historicals)

### Forecast (columns J-N)
- Revenue: `={COL}${REV_SCHEDULE_ROW}` (green, pulls from Revenue Schedule)
- COGS: `={COL}${COGS_SCHEDULE_ROW}` (green, pulls from Cost Schedule)
- Gross Profit: `={COL}{IS_REV}+{COL}{IS_COGS}` (costs negative)
- SGA: `={COL}${SGA_SCHEDULE_ROW}` (green)
- Other: `={COL}${OTHER_SCHEDULE_ROW}` (green)
- EBITDA: `={COL}{GROSS}+{COL}{IS_SGA}+{COL}{IS_OTHER}` (costs negative)
- EBITDA Check: `={COL}{IS_REV}+{COL}{IS_COGS}+{COL}{IS_SGA}+{COL}{IS_OTHER}`
- D&A: `={COL}${DA_TOTAL_ROW}` (green -> Step 9, negative value)
- EBIT: `={COL}{EBITDA}+{COL}{DA}` (D&A negative reduces EBITDA)
- Interest: `={COL}${INTEREST_ROW}` (green -> Step 10, negative)
- EBT: `={COL}{EBIT}+{COL}{INTEREST}` (Interest negative)
- Tax: `={COL}${TOTAL_TAX_ROW}` (green -> Step 11, negative)
- Net Income: `={COL}{EBT}+{COL}{TAX}` (Tax negative)

### Forward References
D&A (Step 9), Interest (Step 10), and Tax (Step 11) are forward references. Until those steps are built:
- Write the formulas pointing to the correct row constants
- The audit should flag these as PENDING (not FAILED) until resolved

## Audit Checklist (auditor MUST address EVERY item)

1. The Income Statement contains exactly 12 labeled rows: Revenue, COGS, Gross Profit, SGA, Other Opex, EBITDA, EBITDA Check, D&A, EBIT, Interest Expense, EBT, Tax Expense, Net Income. Scan column E labels and verify all 12 exist.
2. Historical cells (columns F through I) for all 12 IS rows have blue font (`cell.font.color.rgb` contains `"0000CC"`) and non-None comments with source text.
3. Each historical value matches the GAAP Income Statement from the research CSV. Load CSV and compare all 12 line items for all 4 historical years. GAAP values only -- NOT management-adjusted EBITDA (Trap #7).
4. Historical cells remain hardcoded blue values and are NOT overwritten by formulas from Steps 9/10/11 (Trap #13). For columns F-I, verify `cell.value` does NOT start with `=` for Revenue, COGS, SGA, Other, EBITDA, D&A, EBIT, Interest, EBT, Tax, NI.
5. Forecast Revenue cells (columns J through N) contain formulas referencing the Revenue Schedule row (green font cross-reference). Verify `cell.value` starts with `=` and references the Revenue Schedule row number.
6. Forecast COGS cells reference the Cost Schedule COGS row. Forecast SGA cells reference Cost Schedule SGA row. Forecast Other cells reference Cost Schedule Other row. Each must be a formula with a row reference to the correct schedule.
7. Forecast Gross Profit = Revenue + COGS (costs negative). Verify formula adds IS Revenue cell and IS COGS cell in each column.
8. Forecast EBITDA = Gross Profit + SGA + Other. Verify formula string references the Gross Profit, SGA, and Other rows within the IS.
9. EBITDA Check row in forecast columns contains formula `=Rev+COGS+SGA+Other` (all 4 items). The computed value must equal the EBITDA row value. Verify: `abs(ebitda_check_value - ebitda_value) < 0.01` for each forecast column.
10. Forecast D&A cells contain a formula referencing the Depreciation Schedule Total D&A row from Step 9 (forward reference). Verify formula contains `=` and references the correct Model sheet row for Total D&A. Cell has green font.
11. Forecast Interest cells contain a formula referencing the Debt Schedule Interest row from Step 10 (forward reference). Cell has green font.
12. Forecast Tax cells contain a formula referencing the Levered Tax Total row from Step 11 (forward reference). Cell has green font.
13. Forecast EBIT = EBITDA + D&A (D&A negative). Verify formula references IS EBITDA and IS D&A rows.
14. Forecast EBT = EBIT + Interest (Interest negative). Verify formula references IS EBIT and IS Interest rows.
15. Forecast Net Income = EBT + Tax (Tax negative). Verify formula references IS EBT and IS Tax rows.
16. EBITDA row has double bottom border: `cell.border.bottom.style == "double"` for columns F through N.
17. Net Income row has double bottom border: `cell.border.bottom.style == "double"` for columns F through N.
18. All cross-sheet reference cells in forecast columns (Revenue, COGS, SGA, Other, D&A, Interest, Tax) have green font: `cell.font.color.rgb` contains green.
19. All within-sheet formula cells (Gross Profit, EBITDA, EBITDA Check, EBIT, EBT, NI) have black font (default or `"000000"`).

## Known Traps
- **#7: "Adjusted" EBITDA** — EBITDA must equal Revenue - COGS - SGA - Other from GAAP IS. Do not use management-adjusted figures.
- **#13: Historical IS overwritten** — When Steps 9/10/11 write formulas, they MUST only write to FORECAST columns. Historical IS must remain hardcoded GAAP (blue font).
- EBITDA Check not tying due to sign convention mismatch between Revenue and Cost schedules
