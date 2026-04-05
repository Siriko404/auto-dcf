# Step 13: UFCF Dual Method

EBITDA method + Net Income method + reconciliation. Both methods MUST produce identical UFCF.
TWO valid approaches exist (Decomposed vs Simplified). Choose ONE. Do NOT mix components.

## Structure
```
Row {hdr}: "UNLEVERED FREE CASH FLOW" header (bold, bottom border)

--- Method 1: EBITDA Approach ---
Row {m1_label}:  "Method 1: EBITDA Approach"
Row {ebitda}:    EBITDA (green -> IS)
Row {ulev_tax}:  Less: Unlevered Total Tax (green -> Step 12, negative)
Row {capex}:     Less: Capital Expenditures (green -> Step 9, negative)
Row {dnwc}:      Less: Change in NWC (green -> Step 8, negative if WC increases)
Row {ufcf1}:     UFCF Method 1 (double border)
NOTE: If using Approach A (decomposed NI), EBITDA method MUST use Unlevered CURRENT Tax.
      If using Approach B (simplified NI), EBITDA method MUST use Unlevered TOTAL Tax.
      The two methods MUST use CORRESPONDING tax components.

--- Method 2: Net Income Approach ---
TWO valid approaches exist. Choose ONE. Do NOT mix components.

**Approach A (Decomposed — recommended, matches CFI reference model):**
Row {m2_label}:    "Method 2: Net Income Approach"
Row {ni}:          Net Income (green -> IS)
Row {da}:          Add: D&A (green -> Step 9 Total D&A, positive)
Row {def_tax}:     Add: Levered Deferred Tax (green -> Step 11 Deferred, positive add-back)
Row {interest}:    Add: Interest Expense (green -> Debt Schedule, positive)
Row {tax_shield}:  Less: CURRENT-ONLY Tax Shield = -(UnlevCurrent - LevCurrent) (negative)
Row {capex2}:      Less: Capex (green, negative)
Row {dnwc2}:       Less: Change in NWC (green, negative if increase)
Row {ufcf2}:       UFCF Method 2 (double border)

**Approach B (Simplified — zeroes deferred tax, uses full shield):**
Row {m2_label}:    "Method 2: Net Income Approach"
Row {ni}:          Net Income (green -> IS)
Row {da}:          Add: D&A (green -> Step 9 Total D&A, positive)
Row {def_tax}:     Deferred Tax = 0 (hardcoded, full shield compensates)
Row {interest}:    Add: Interest Expense (green -> Debt Schedule, positive)
Row {tax_shield}:  Less: FULL Tax Shield = -(UnlevTotal - LevTotal) (negative)
Row {capex2}:      Less: Capex (green, negative)
Row {dnwc2}:       Less: Change in NWC (green, negative if increase)
Row {ufcf2}:       UFCF Method 2 (double border)

--- Reconciliation ---
Row {recon}:     Reconciliation = Method 1 - Method 2 (MUST = 0)
```

## Why Decomposed (not Interest*(1-T))

The common shortcut `Interest*(1-T)` is equivalent to `+Interest - Tax Shield` ONLY when:
- There are no tax losses
- There is no deferred tax

With deferred tax and loss carryforwards, the shortcut breaks. The decomposed form is:
```
NI + D&A + Deferred Tax + Interest - Tax Shield - Capex - ΔNWC
```
This reconciles to the EBITDA method in ALL cases.

## Key Formulas

### Approach A (Decomposed):
- UFCF1: `={EBITDA}+{ULEV_CURRENT_TAX}+{CAPEX}+{DNWC}` (current tax only, negative)
- UFCF2: `={NI}+{DA}+{LEV_DEFERRED_TAX}+{INTEREST}-({ULEV_CURRENT}-{LEV_CURRENT})+{CAPEX}+{DNWC}`
- Recon: `={UFCF1}-{UFCF2}` = 0

### Approach B (Simplified):
- UFCF1: `={EBITDA}+{ULEV_TOTAL_TAX}+{CAPEX}+{DNWC}` (total tax, negative)
- UFCF2: `={NI}+{DA}+0+{INTEREST}-{FULL_TAX_SHIELD}+{CAPEX}+{DNWC}`
- Recon: `={UFCF1}-{UFCF2}` = 0

### CRITICAL: Do NOT mix approaches
Using Total tax in EBITDA method + Levered Deferred in NI method + Full Tax Shield = WILL NOT RECONCILE. This was the #1 bug in the first live execution (4 fix iterations wasted).

## Audit Checklist
1. EBITDA Method uses Unlevered CURRENT tax (not Total) if Approach A chosen — verify cell formula references Step 12 Current row, not Total row (Trap #22)
2. NI Method includes Levered Deferred Tax add-back — verify cell formula references Step 11 Deferred Tax row with positive sign (or hardcoded 0 if Approach B)
3. NI Method subtracts Tax Shield — verify cell formula contains a negative term referencing Unlevered minus Levered current tax (Approach A) or total tax (Approach B)
4. Tax Shield = Unlevered Total - Levered Total (positive value) — read Tax Shield cells and confirm the subtraction direction produces a positive number that is then negated in the UFCF formula
5. Reconciliation row = Method 1 - Method 2, must equal zero for ALL forecast columns — read every forecast-period reconciliation cell with openpyxl and assert abs(value) < 0.01
6. All component formulas reference correct row constants — read each UFCF component cell formula string and verify row references match the defined row constants for EBITDA, Capex, DNWC, NI, D&A, Interest, etc.
7. Font colors correct — verify green font (RGB 006100 or theme equivalent) on all cells that pull from other schedules (EBITDA from IS, Capex from Step 9, DNWC from Step 8, NI from IS, D&A from Step 9, Interest from Debt Schedule)
8. Double borders on M1 Total (UFCF Method 1) and M2 Total (UFCF Method 2) rows — read border.bottom.style and confirm "double" for every cell in those rows across all period columns
9. Source comments on any CSV-derived values — check for cell comments on historical-period cells that trace back to parsed CSV filing data
10. Spot-check: manually verify reconciliation algebra — pick one forecast column, read all component values, recompute both methods in Python, and confirm they match within $0.01

## Known Traps
- **#19: Deferred tax missing** — reconciliation will fail
- Using Interest*(1-T) shortcut instead of decomposed form
- Wrong sign on ΔNWC (increase in WC = cash outflow = negative)
- Tax Shield pulled from wrong schedule (must be Step 12, not Step 11)
- Reconciliation off by deferred tax amount = smoking gun for #19
- **#22: Tax component mismatch** — Using Total tax in EBITDA + Deferred in NI + Full Shield. Components must be from the SAME approach. This caused 4 failed iterations in the first live test.
