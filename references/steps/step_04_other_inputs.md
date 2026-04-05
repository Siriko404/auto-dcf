# Step 4: Other Inputs (Inputs Sheet)

Dates, opening balances, rates, depreciation parameters, and per-year assumptions. This section provides the non-driver inputs that multiple downstream schedules reference.

## Structure
```
Row {hdr}:       "OTHER MODEL INPUTS" header (bold, bottom border)
Row {blank}:     (blank)

--- Key Dates ---
Row {date_lbl}:  "Key Dates"
Row {val_date}:  Valuation Date (blue — in F column)
Row {cf_dates}:  Cash Flow Dates (blue — one per forecast col J-N)

--- Opening Balances ---
Row {bal_lbl}:   "Opening Balances"
Row {ppe}:       PP&E Net Book Value (blue — from most recent audited BS)
Row {ar}:        Accounts Receivable (blue)
Row {inv}:       Inventory (blue)
Row {ap}:        Accounts Payable (blue)
Row {debt}:      Total Debt (blue)
Row {cash}:      Cash & Equivalents (blue)
Row {lev_tl}:    Levered Tax Loss Carryforward (blue — 0 if none)
Row {ulev_tl}:   Unlevered Tax Loss Carryforward (blue — 0 if none)

--- Rates & Assumptions ---
Row {rate_lbl}:  "Rates & Assumptions"
Row {tax_rate}:  Statutory Tax Rate (blue — from tax notes, NOT effective rate)
Row {useful_life}: Useful Life PP&E in years (blue — e.g., 10, NOT 0.265)
Row {acct_depr}: Accounting Depreciation Rate (blue — typically 1/Useful Life)
Row {cca_rate}:  Tax Depreciation Rate / CCA Rate (blue — from tax notes)
Row {debt_rate}: Debt Interest Rate (blue — from credit facility terms)
Row {tgr}:       Terminal Growth Rate (blue)
Row {exit_mult}: Exit EBITDA Multiple (blue)

--- Year-by-Year Assumptions ---
Row {yr_lbl}:    "Year-by-Year Assumptions"
Row {shares}:    Shares Outstanding per year (blue — one per col J-N)
```

## CRITICAL: Depreciation Rate Rows

Two separate depreciation rate rows are REQUIRED:

1. **Row {acct_depr}: Accounting Depreciation Rate** — Used by Step 9 (Depreciation Schedule) for book depreciation. Typically `= 1 / Useful Life` (e.g., 10% for 10-year life). This drives GAAP D&A.

2. **Row {cca_rate}: Tax/CCA Depreciation Rate** — Used by Steps 11-12 (Tax Schedules) for tax depreciation / Capital Cost Allowance. Often higher than accounting rate (accelerated). This drives deferred tax timing differences.

These MUST be on separate, clearly labeled rows. The accounting rate feeds the Depreciation Schedule. The CCA rate feeds the Tax Schedules. Mixing them up is Mistake #1.

## Key Formulas
- Accounting Depreciation Rate: `= 1 / {useful_life}` or hardcoded if company-specific
- Cash Flow Dates: Hardcoded date values (July 1 of each forecast year for mid-year convention)
- All other cells: blue hardcoded values from research file

## Audit Checklist (auditor MUST address EVERY item)

1. Valuation Date cell (column F) contains a date value. Verify via `cell.is_date` or `isinstance(cell.value, datetime)`.
2. Cash Flow Date cells (columns J through N) each contain a date value where month=7 and day=1 (mid-year July 1 convention). Read each `cell.value` and confirm `.month == 7` and `.day == 1`.
3. PP&E Net Book Value opening balance cell has blue font and its value matches the CSV source data exactly (Trap #24: must EXCLUDE ROU assets -- verify the comment references PP&E excluding right-of-use).
4. All opening balance cells (PP&E, AR, Inventory, AP, Debt, Cash) have blue font (`cell.font.color.rgb` contains `"0000CC"`) and non-None comments (`cell.comment is not None`).
5. Each opening balance value matches the corresponding value in the research CSV file to the dollar. Load CSV and compare programmatically.
6. Cash opening balance exists as a dedicated input cell (not omitted here and hardcoded later in DCF bridge -- Trap #12).
7. Statutory Tax Rate cell has blue font, value between 0.20 and 0.35 (reasonable range), and comment text does NOT contain "effective" -- must reference statutory/marginal rate (26% for Canada, confirm from tax notes).
8. Useful Life cell value is a whole number between 5 and 20 (years). It must NOT be a decimal like 0.265 (that would be a rate, not a life). Verify `isinstance(cell.value, (int, float)) and cell.value >= 5`.
9. Accounting Depreciation Rate row exists with a label containing "Accounting" or "Acct" and "Depreciation" or "Depr". Its value should approximately equal `1 / Useful Life`. Verify the difference is < 0.01.
10. Tax/CCA Depreciation Rate row exists as a SEPARATE row from Accounting Depreciation Rate (Trap #1). Scan column E labels; confirm two distinct rows with different labels, one containing "CCA" or "Tax" and the other containing "Accounting" or "Acct".
11. The CCA Rate value differs from the Accounting Depreciation Rate value (they are almost never identical -- flag if equal as likely Trap #1 error).
12. Debt Interest Rate cell has blue font and a comment referencing credit facility terms (not assumed).
13. Terminal Growth Rate (TGR) cell exists with blue font and a value typically between 0.01 and 0.04.
14. Exit EBITDA Multiple cell exists with blue font and a value typically between 4.0 and 20.0.
15. Shares Outstanding cells exist in forecast columns (J through N) with blue font. All values should be positive and within 10% of each other (consistency check, Trap #10).
16. Levered Tax Loss and Unlevered Tax Loss carryforward cells exist (value may be 0 if not applicable).
17. Every blue-font input cell in this section has `cell.comment is not None` with non-empty comment text containing a traceable source reference.
18. The Accounting Depreciation Rate row and CCA Rate row are clearly separated (not adjacent without labels) -- verify at least one of them has an explicit label in column E distinguishing it from the other.

## Known Traps
- **#1 ROOT CAUSE: Acct Depr Rate and Tax/CCA Rate on adjacent rows** — Label them CLEARLY. Step 9 uses accounting rate. Steps 11-12 use CCA rate. Mixing them is catastrophic.
- **#8: Wrong PP&E value** — Must match audited BS. Compare to research file exactly (e.g., $44.8M, not $85M).
- **#10: Shares outstanding inconsistency** — One source cell, referenced everywhere.
- **#12: Cash hardcoded in DCF bridge** — Cash must be in this section so the bridge references it.
- Tax rate mix-up (effective vs statutory) — Effective rate varies year to year; statutory is the legal rate.
- **#19: Deferred tax impossible without both depreciation rates** — If CCA rate is missing, Steps 11-12 cannot compute deferred tax, and UFCF reconciliation will fail.
