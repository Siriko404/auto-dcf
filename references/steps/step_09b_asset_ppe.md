# Step 9b: Asset Schedule — PP&E Summary Corkscrew

The 12th schedule. Summarizes total PP&E from the Existing and New Asset sub-corkscrews in Step 9 into one consolidated PP&E corkscrew. This is the balance sheet-ready view.

## Structure
```
Row {hdr}:     "ASSET SCHEDULE (PP&E)" header (bold, bottom border)
Row {blank}:   (blank)
Row {open}:    PP&E Opening NBV
Row {capex}:   Capital Expenditures (positive addition)
Row {depr}:    Total PP&E Depreciation (negative, from Step 9)
Row {close}:   PP&E Closing NBV (double border)
Row {blank}:   (blank)
Row {check}:   PP&E Check = Closing - (Existing Close + New Close) [MUST = 0]
```

## Key Formulas

### Historical (columns F-I)
- Opening, Capex, Depreciation, Closing: hardcoded from BS / CF statement, BLUE font

### Forecast (columns J-N)
- Opening: `={PREV}${PPE_CLOSE}` (prior year PP&E Closing from this schedule)
- Capex: `={COL}${NEW_CAPEX_ROW}` (green -> Step 9 New Capex row)
- Depreciation: `={COL}${DEPR_TOT_ROW}` (green -> Step 9 Total PP&E Depreciation, negative)
- Closing: `={COL}{PPE_OPEN}+{COL}{PPE_CAPEX}+{COL}{PPE_DEPR}`

### Cross-Check
- `={COL}{PPE_CLOSE}-({COL}${EX_CLOSE}+{COL}${NEW_CLOSE})`
- Must equal 0 — confirms the summary ties to the detail sub-corkscrews

### First Forecast Column (J)
- Opening: `={I}${PPE_CLOSE}` (last historical PP&E Closing)
  - Must match Inputs!{ppe_opening} (the opening balance in Other Inputs)

## Audit Checklist (auditor MUST address EVERY item)

1. Historical PP&E cells (columns F through I) for Opening, Capex, Depreciation, and Closing all have blue font (`cell.font.color.rgb` contains `"0000CC"`) and non-None comments with source text referencing audited BS or CF statement.
2. Each historical PP&E value matches the audited balance sheet / cash flow statement from the research CSV. Load CSV and compare programmatically (Trap #8: must be exact match).
3. First forecast Opening cell (column J) contains a formula referencing the last historical Closing cell (column I, Closing row). Verify `cell.value` starts with `=` and references the correct column/row.
4. For forecast columns K through N, each Opening cell contains a formula referencing the prior column's Closing cell from THIS schedule (not Step 9 sub-corkscrew Closing -- Trap #2 analogy). Verify formula row reference matches this schedule's Closing row number.
5. Forecast Capex cells (columns J through N) each contain a formula referencing the Step 9 New Capex row. Verify formula string contains `=` and the row number matches the New Capex row from the Depreciation Schedule. Capex values must be positive (adds to NBV).
6. Forecast Depreciation cells (columns J through N) each contain a formula referencing the Step 9 Total PP&E Depreciation row. Verify formula references the correct row. Values must be negative (reduces NBV): `cell_value < 0` for each forecast column.
7. Forecast Capex and Depreciation cells have green font (`cell.font.color.rgb` contains green) since they are cross-schedule references within the Model sheet.
8. Forecast Closing cells contain the corkscrew formula: Opening + Capex + Depreciation. Verify formula string references exactly the Opening, Capex, and Depreciation rows for the same column.
9. Corkscrew integrity check: for each forecast column, read the computed values and verify `abs(Opening + Capex + Depreciation - Closing) < 0.01`.
10. PP&E Check row exists and its formula computes `Closing - (Existing Close + New Close)` referencing the Step 9 sub-corkscrew closing rows. Verify the computed value equals 0 (or `abs(value) < 0.01`) for every forecast column.
11. Closing row has double bottom border: `cell.border.bottom.style == "double"` for each cell in columns F through N.
12. Forecast Closing formula cells have black font (default or `"000000"`), not green (they are within-schedule calculations).
13. This schedule is the 12th schedule header on the Model sheet. Count all bold header rows with bottom borders above this one; confirm this is number 12.
14. The first forecast Opening value ties to the Inputs sheet PP&E opening balance. Load the Inputs PP&E NBV cell value and verify it equals the first forecast Opening value: `abs(inputs_ppe - forecast_opening) < 0.01`.

## Known Traps
- **#2 (by analogy): Opening references wrong row** — Must reference this schedule's own Closing row, not Step 9's sub-corkscrew closing
- **#8: Wrong PP&E value** — Historical PP&E must match audited BS exactly
- Depreciation sign: must be negative in this schedule (reduces NBV)
- Capex sign: must be positive (adds to NBV)
- Not building this schedule at all — it is the 12th schedule required by institutional standards
