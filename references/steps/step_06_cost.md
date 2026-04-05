# Step 6: Cost Schedule (Model Sheet)

COGS, SGA, and Other Operating Expenses. Historical GAAP values plus forecast linked to percentage drivers from Inputs.

## Structure
```
Row {hdr}:       "COST SCHEDULE" header (bold, bottom border)
Row {blank}:     (blank)
Row {cogs}:      COGS (Cost of Goods Sold / Cost of Revenue)
Row {cogs_pct}:  COGS as % of Revenue
Row {sga}:       SGA (Selling, General & Administrative)
Row {sga_pct}:   SGA as % of Revenue
Row {other}:     Other Operating Expenses
Row {other_pct}: Other Opex as % of Revenue
Row {total}:     Total Operating Costs (double border, sum of all cost items)
```

## Key Formulas

### Historical (columns F-I)
- COGS, SGA, Other: hardcoded GAAP values, BLUE font, source comments
- Percentage rows: `=-{COL}{COST}/{COL}{REV}` (display as positive %)
- Total: `={COL}{COGS}+{COL}{SGA}+{COL}{OTHER}`

### Forecast (columns J-N)
- COGS: `=-{COL}${REV}*Inputs!{COL}${COGS_ACTIVE}`
  (stored as NEGATIVE; Revenue * COGS% gives positive cost, negate for sign convention)
  OR: `={COL}${REV}*Inputs!{COL}${COGS_ACTIVE}` with explicit positive convention — PICK ONE and be consistent
- SGA: `=-{COL}${REV}*Inputs!{COL}${SGA_ACTIVE}` (same pattern)
- Other: `=-{COL}${REV}*Inputs!{COL}${OTHER_ACTIVE}` (same pattern)
- Pct rows: `=-{COL}{COGS}/{COL}{REV}` (display as positive %)
- Total: `={COL}{COGS}+{COL}{SGA}+{COL}{OTHER}`

### Sign Convention
Choose ONE convention and enforce across the entire model:
- **Option A (recommended)**: Costs stored NEGATIVE. Gross Profit = Revenue + COGS. EBITDA = Revenue + COGS + SGA + Other.
- **Option B**: Costs stored POSITIVE. Gross Profit = Revenue - COGS. EBITDA = Revenue - COGS - SGA - Other.

## Audit Checklist (auditor MUST address EVERY item)

1. Historical COGS cells (columns F through I) have blue font (`cell.font.color.rgb` contains `"0000CC"`) and each `cell.comment is not None` with source text.
2. Historical SGA cells (columns F through I) have blue font and non-None comments with source text.
3. Historical Other Operating Expenses cells (columns F through I) have blue font and non-None comments with source text.
4. Each historical cost value matches the GAAP Income Statement from the research CSV. Load CSV and compare COGS, SGA, Other for all 4 historical years. Values stored as NEGATIVE (expense convention).
5. Forecast COGS cells (columns J through N) each contain a formula referencing BOTH the Revenue row on the Model sheet AND the COGS% ACTIVE row on the Inputs sheet. Parse formula string: must contain a cross-sheet reference `Inputs!` and a same-sheet Revenue row reference.
6. Forecast SGA cells (columns J through N) each contain a formula referencing Revenue and the SGA% ACTIVE row on Inputs. Same verification as item 5.
7. Forecast Other Opex cells (columns J through N) each contain a formula referencing Revenue and the Other% ACTIVE row on Inputs. Same verification as item 5.
8. Confirm forecast cost formulas link to ACTIVE driver rows (not Best/Base/Worst scenario rows). Extract the Inputs row number from each formula and verify it matches the known Active row number for that driver group.
9. Sign convention is consistent: all COGS, SGA, and Other cells (historical and forecast) are either all negative or all positive. Read values for all columns F-N and confirm `all(v < 0 for v in values)` or `all(v > 0 for v in values)`.
10. Percentage display rows (COGS%, SGA%, Other%) contain formulas that divide the negative cost by Revenue and negate (producing a positive percentage). Verify each cell value starts with `=` and the computed result is positive (between 0.0 and 1.0).
11. Percentage row cells have number format `"0.0%"` -- check `cell.number_format` for each cell in columns F through N.
12. Cost amount rows have number format containing `"#,##0"` or `"#,##0.0"` -- check `cell.number_format`.
13. Total Operating Costs row contains a formula summing COGS + SGA + Other for each column. Verify formula references exactly the 3 cost rows.
14. Total Operating Costs row has a double bottom border: `cell.border.bottom.style == "double"` for each cell in columns F through N.
15. All forecast formula cells referencing other sheets (Revenue from Model, drivers from Inputs) have green font: `cell.font.color.rgb` contains `"00FF00"` or equivalent green.
16. All non-cross-sheet formula cells have black font (default or `"000000"`).

## Known Traps
- Sign convention inconsistency (some costs positive, some negative) — cascades to IS, EBITDA, and every downstream schedule
- Wrong Inputs row for COGS% vs SGA% (off-by-one in row constants)
- If COGS%/SGA% are per-year direct inputs (not CHOOSE-switched), link to the correct Other Inputs rows instead of driver ACTIVE rows
