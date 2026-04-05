# Step 9: Depreciation Schedule

THE most bug-prone schedule. Two sub-corkscrews (Existing + New Assets) with half-year convention.

## Structure
```
Row {hdr}: "DEPRECIATION SCHEDULE" header (bold, bottom border)

--- Existing Assets Sub-Corkscrew ---
Row {ex_open}:  Existing Assets Opening NBV
Row {ex_depr}:  Existing Assets Depreciation = -Opening * Acct Depr Rate
Row {ex_close}: Existing Assets Closing = Opening + Depreciation

--- New Assets Sub-Corkscrew ---
Row {new_open}:  New Assets Opening NBV (= prior year New Closing; year 1 = 0)
Row {new_capex}: Capital Expenditures (from Inputs: Revenue * Capex%)
Row {new_depr}:  New Assets Depreciation (half-year on capex, full on opening)
Row {new_close}: New Assets Closing = Opening + Capex + Depreciation

--- Summary ---
Row {depr_tot}: Total PP&E Depreciation = Existing Depr + New Depr (double border)
Row {da_other}: Other D&A (ROU, Intangibles — flat or from Inputs)
Row {da_total}: Total D&A = PP&E Depr + Other D&A (links to IS)
```

## CRITICAL Formulas

### Existing Assets
- Opening: `={PREV}${EX_CLOSE}` — MUST reference CLOSING, NOT depreciation row
- Depreciation: `=-{COL}${EX_OPEN}*Inputs!$F${ACCT_DEPR_RATE}`
  - Uses ACCOUNTING depreciation rate (e.g., 10% = 1/10yr useful life)
  - NOT the tax/CCA rate
  - NOT the useful life directly (use rate = 1/life, pre-computed in Inputs)
- Closing: `={COL}${EX_OPEN}+{COL}${EX_DEPR}`

### New Assets — HALF-YEAR CONVENTION
- Opening: `={PREV}${NEW_CLOSE}` (first forecast year = 0)
- Capex: `={COL}${REV_ROW}*Inputs!{COL}${CAPEX_ACTIVE}` (or direct from Inputs)
- Depreciation:
  ```
  =-{COL}${NEW_OPEN}*Inputs!$F${ACCT_DEPR_RATE}
   -{COL}${NEW_CAPEX}*Inputs!$F${ACCT_DEPR_RATE}*0.5
  ```
  **First term**: Full-year depreciation on opening NBV (prior years' accumulated capex)
  **Second term**: HALF-year depreciation on current-year capex (0.5 multiplier)
  Standard accounting: assets acquired mid-year get 50% depreciation in year of acquisition.
- Closing: `={COL}${NEW_OPEN}+{COL}${NEW_CAPEX}+{COL}${NEW_DEPR}`

### NBV Floor Guard
Closing NBV formulas should use MAX(0,...) to prevent negative book value:
- Existing Close: `=MAX(0, {COL}${EX_OPEN}+{COL}${EX_DEPR})`
- New Close: `=MAX(0, {COL}${NEW_OPEN}+{COL}${NEW_CAPEX}+{COL}${NEW_DEPR})`

Without this guard, assets can have negative NBV in extended forecasts, which produces nonsensical positive depreciation on negative balances.

### Historical Population
Populate historical columns (F-I) with actual PP&E NBV data (blue font) from the audited balance sheet. Do NOT leave historical depreciation/PP&E rows empty.

### Summary
- Total PP&E Depr: `={COL}${EX_DEPR}+{COL}${NEW_DEPR}`
- Total D&A: `={COL}${DEPR_TOT}+{COL}${DA_OTHER}`

## Audit Checklist (auditor MUST address EVERY item)
- [ ] 1. Two separate sub-corkscrews exist: Existing Assets (rows {ex_open} to {ex_close}) and New Assets (rows {new_open} to {new_close})
- [ ] 2. Existing Opening J{ex_open} = Inputs!$F${PPE_NBV} (first forecast, green font) — verify actual cell ref
- [ ] 3. Existing Opening K{ex_open} = J{ex_close} — references CLOSING row, NOT depreciation (Trap #2)
- [ ] 4. Existing Depr uses Inputs!$F${ACCT_DEPR_RATE}, NOT Inputs!$F${CCA_RATE} or $F${TAX_RATE} (Trap #1)
- [ ] 5. New asset depr formula contains `*0.5` on the capex term (half-year convention, Trap #20)
- [ ] 6. New Opening J{new_open} = 0 (blue font, hardcoded) for first forecast year
- [ ] 7. Capex sign: positive in formula. Depreciation sign: negative in formula
- [ ] 8. Existing Closing = Opening + Depreciation (verify algebra)
- [ ] 9. New Closing = Opening + Capex + Depreciation (verify algebra)
- [ ] 10. Total PP&E Depr = Existing Depr + New Depr, has double-bottom border
- [ ] 11. Total D&A = -Total PP&E Depr + Other D&A (positive value for IS linkage)
- [ ] 12. Historical columns (F-I) have blue-font values from CSV, with source comments
- [ ] 13. All source comments reference CSV row (e.g., "CSV: financials.csv row 45")
- [ ] 14. Font colors: blue inputs, black formulas, green cross-sheet refs
- [ ] 15. Spot-check: manually compute J{ex_depr} = -PP&E_NBV * acct_depr_rate, compare to cell

## Known Traps
- **#1 CATASTROPHIC: Depreciation / Tax Rate instead of * Acct Rate** — Adjacent rows in Inputs. Using CCA rate instead of accounting rate produces wrong depreciation.
- **#2 CATASTROPHIC: Opening = Depreciation row instead of Closing** — Arithmetic offsets `r+1` when depreciation sits between Opening and Closing.
- **#20: Missing half-year convention** — Full year on new capex overstates depreciation by ~50% of capex * rate.
- Forgetting to set New Opening = 0 for first forecast year
