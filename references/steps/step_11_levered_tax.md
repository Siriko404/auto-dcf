# Step 11: Levered Tax Schedule

Current AND Deferred tax from EBT. Deferred tax is REQUIRED for UFCF reconciliation.

## Structure
```
Row {hdr}: "TAX SCHEDULE — LEVERED" header (bold, bottom border)

--- Current Tax ---
Row {ebt}:         EBT (green -> IS)
Row {tl_open}:     Tax Loss CF Opening
Row {tl_used}:     Tax Loss Used = MIN(Opening, MAX(0, EBT))
Row {taxable}:     Taxable Income = MAX(0, EBT) - Tax Loss Used
Row {current_tax}: Current Tax = Taxable * Statutory Rate (negative)
Row {tl_close}:    Tax Loss CF Closing = Opening - Used (+ new losses if EBT < 0)

--- Deferred Tax ---
Row {acct_depr}:   Accounting Depreciation (green -> Depreciation Schedule Total Depr)
Row {tax_depr}:    Tax/CCA Depreciation = UCC Opening * CCA Rate (from Inputs)
Row {timing}:      Timing Difference = Accounting Depr - Tax Depr
Row {deferred}:    Deferred Tax = Timing * Statutory Rate

--- Total ---
Row {total_tax}:   Total Tax = Current + Deferred (double border)
```

### Optional: UCC (Undepreciated Capital Cost) Corkscrew
For Canadian companies using CCA (Capital Cost Allowance):
Row {ucc_open}:  UCC Opening (tax basis of assets)
Row {ucc_cca}:   CCA = UCC * CCA Rate + Capex * CCA Rate * 0.5 (half-year rule)
Row {ucc_close}: UCC Closing = Opening + Capex - CCA

Tax Depreciation should reference UCC-based CCA, not PP&E * CCA Rate.
The difference: UCC tracks the declining tax basis; PP&E tracks the accounting basis. They diverge over time.

## Why Deferred Tax is Required

Without deferred tax, the UFCF NI method cannot reconcile with the EBITDA method.
- Net Income includes deferred tax effects (accounting depreciation hits NI via D&A)
- EBITDA method uses unlevered current tax only
- The difference = deferred tax, which the NI method must add back

**If you skip deferred tax, the reconciliation check (Step 13) will NEVER be zero.**

## Key Formulas
- Current Tax: `=IF({COL}${TAXABLE}>0, -{COL}${TAXABLE}*Inputs!$F${TAX_RATE}, 0)`
- Tax Loss Used: `=MIN({COL}${TL_OPEN}, MAX(0, {COL}${EBT}))`
- Acct Depr: `={COL}${DEPR_TOTAL_ROW}` (green — from Depreciation Schedule)
- Tax Depr: `=-{COL}${UCC_OPEN}*Inputs!$F${CCA_RATE}` (if UCC corkscrew exists, else simplified)
- Timing: `={COL}${ACCT_DEPR}-{COL}${TAX_DEPR}` (sign: both are negative, so difference may be positive or negative)
- Deferred Tax: `={COL}${TIMING}*Inputs!$F${TAX_RATE}`
- Total Tax: `={COL}${CURRENT_TAX}+{COL}${DEFERRED}` (both negative)

## Audit Checklist
1. EBT row references IS correctly — read the EBT cell formula for each period and confirm it points to the Income Statement EBT row (e.g., `=J$XX` where XX is the IS EBT row)
2. Tax Loss CF corkscrew: Opening = prior Closing — for every forecast column, read TL Opening cell formula and verify it references the prior column's TL Closing cell (e.g., `=I$YY` where YY is the Closing row)
3. Tax Loss Used = MIN(Opening, MAX(0, EBT)) — read the Tax Loss Used cell formula string and confirm it contains MIN/MAX logic matching this exact pattern
4. Current Tax = Taxable * Statutory Rate (from Inputs) — read Current Tax cell formula and verify it multiplies the Taxable Income row by the statutory rate cell on Inputs (e.g., `Inputs!$F$XX`), NOT an effective rate
5. Deferred Tax present (Trap #19) — confirm Deferred Tax row exists and is non-empty; verify cell formula references the Timing Difference row multiplied by the statutory rate; absence means UFCF reconciliation in Step 13 will fail
6. Total Tax = Current + Deferred — read Total Tax cell formula and confirm it sums exactly the Current Tax row and Deferred Tax row for each period column
7. Historical total tax matches CSV values — for each historical column, read the Total Tax cell value and compare against the parsed CSV filing data; flag any discrepancy > $0.01
8. All font colors correct — verify green font (RGB 006100 or theme equivalent) on EBT cells (from IS), Accounting Depreciation cells (from Depreciation Schedule), and any other cross-schedule references
9. Double border on Total Tax row — read border.bottom.style for every cell in the Total Tax row across all period columns and confirm "double"
10. Spot-check computation — pick one forecast column, read EBT, TL Opening, TL Used, Taxable, Current Tax, Acct Depr, Tax Depr, Timing, Deferred Tax, and Total Tax values; recompute each in Python and confirm all match within $0.01

## Known Traps
- **#19 CRITICAL: Deferred tax omitted** — UFCF reconciliation will fail
- Using effective rate instead of statutory
- No floor on taxable income (allowing negative tax)
- Tax loss carryforward not rolling correctly
