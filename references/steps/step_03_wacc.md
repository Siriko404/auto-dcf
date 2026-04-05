# Step 3: WACC Section (Inputs Sheet)

5 real peer companies, Hamada beta unlevering/re-levering, CAPM with CRP, and WACC calculation. All peer data must be sourced from actual market data.

## Structure
```
Row {hdr}:       "WACC CALCULATION" header (bold, bottom border)
Row {blank}:     (blank)
Row {peer_lbl}:  "Comparable Companies"
Row {peer_cols}: Column headers: Company | Mkt Cap | Debt | Cash | D/E | Lev Beta | Tax Rate | Unlev Beta

Row {peer1}:     Peer 1 data (blue inputs + formulas)
Row {peer2}:     Peer 2
Row {peer3}:     Peer 3
Row {peer4}:     Peer 4
Row {peer5}:     Peer 5

Row {avg}:       Average Unlevered Beta = AVERAGE(peer1..peer5 unlev beta)
Row {med}:       Median Unlevered Beta = MEDIAN(peer1..peer5 unlev beta)

Row {blank}:     (blank)
Row {beta_lbl}:  "Beta Calculation"
Row {sel_bu}:    Selected Unlevered Beta (blue — chosen from avg/med)
Row {tgt_de}:    Target D/E Ratio (blue)
Row {marg_t}:    Marginal Tax Rate (blue)
Row {relev_b}:   Re-levered Beta = B_U * (1 + (1-T) * D/E)

Row {blank}:     (blank)
Row {capm_lbl}:  "Cost of Equity (CAPM)"
Row {rf}:        Risk-Free Rate Rf (blue)
Row {erp}:       Equity Risk Premium ERP (blue)
Row {crp}:       Country Risk Premium CRP (blue)
Row {size}:      Size Premium (blue, 0 if not used)
Row {beta}:      Beta (green -> re-levered beta above)
Row {re}:        Cost of Equity Re = Rf + CRP + ERP*Beta + Size

Row {blank}:     (blank)
Row {debt_lbl}:  "Cost of Debt"
Row {rf2}:       Rf (green -> Rf above)
Row {spread}:    Credit Spread (blue — from facility terms)
Row {kd_pre}:    Pre-tax Kd = Rf + Spread
Row {tax2}:      Tax Rate (green -> marginal tax above)
Row {kd_post}:   After-tax Kd = Kd * (1-T)

Row {blank}:     (blank)
Row {wacc_lbl}:  "WACC"
Row {we}:        Equity Weight We (blue)
Row {wd}:        Debt Weight Wd = 1 - We
Row {re2}:       Re (green -> Cost of Equity above)
Row {rd2}:       Rd(1-T) (green -> After-tax Kd above)
Row {wacc}:      WACC = We*Re + Wd*Rd(1-T) (double border)
```

## Key Formulas

### Peer Unlevering (Hamada)
```
Unlevered Beta = Levered Beta / (1 + (1 - Tax) * D/E)
=F{lev_beta} / (1 + (1 - F{tax}) * F{de})
```

### Re-levering (Hamada)
```
Re-levered Beta = B_U * (1 + (1 - T) * D/E)
=F{sel_bu} * (1 + (1 - F{marg_t}) * F{tgt_de})
```

### CAPM (with CRP)
```
Re = Rf + CRP + ERP * Beta + Size
=F{rf} + F{crp} + F{erp} * F{beta} + F{size}
```

### After-tax Cost of Debt
```
Kd(1-T) = (Rf + Spread) * (1 - T)
=F{kd_pre} * (1 - F{tax2})
```

### WACC
```
WACC = We * Re + Wd * Rd(1-T)
=F{we} * F{re2} + F{wd} * F{rd2}
```

### Peer Statistics — MUST be Excel formulas
```
=AVERAGE(F{peer1_bu}:F{peer5_bu})
=MEDIAN(F{peer1_bu}:F{peer5_bu})
```

## Audit Checklist (auditor MUST address EVERY item)

1. Exactly 5 peer company rows exist in the comparable companies table. Scan rows below the "Comparable Companies" label until a blank row; count must equal 5.
2. Each peer row has non-empty values in all data columns (Mkt Cap, Debt, Cash, D/E, Levered Beta, Tax Rate, Unlevered Beta). No cell in these columns may be None or empty string.
3. Every peer data input cell (Mkt Cap, Debt, Cash, Levered Beta, Tax Rate) has blue font: `cell.font.color.rgb` contains `"0000CC"`.
4. Every blue-font peer input cell has a non-None comment (`cell.comment is not None`) whose text references a real data source (e.g., ticker, filing date, or data provider).
5. Each peer's D/E ratio cell contains a formula (starts with `=`) that divides Debt by Mkt Cap -- not a hardcoded number. Verify `cell.value` is a string starting with `=`.
6. Each peer's Unlevered Beta cell contains a Hamada unlevering formula: the formula string must contain a division by `(1+` and reference the peer's Levered Beta, Tax Rate, and D/E cells. Confirm direction: B_U = B_L / (1 + (1-T) * D/E).
7. The Average Unlevered Beta cell contains a formula starting with `=AVERAGE(` that references the range spanning all 5 peer Unlevered Beta cells. Must be an Excel formula, NOT a hardcoded float (Trap #15).
8. The Median Unlevered Beta cell contains a formula starting with `=MEDIAN(` that references the range spanning all 5 peer Unlevered Beta cells. Must be an Excel formula, NOT a hardcoded float (Trap #15).
9. The Re-levered Beta cell contains a Hamada re-levering formula: formula string must contain multiplication by `(1+` and reference Selected Unlevered Beta, Marginal Tax Rate, and Target D/E. Confirm direction: B_L = B_U * (1 + (1-T) * D/E).
10. The Cost of Equity (Re) cell contains a CAPM formula with all 4 terms: Rf, CRP, ERP*Beta, and Size Premium. Verify the formula string contains addition of at least 4 cell references (Rf + CRP + ERP*Beta + Size).
11. The CRP cell exists and has a value (even if 0.0 for domestic companies). Confirm cell is not None.
12. The Pre-tax Kd cell contains a formula summing Rf and Credit Spread. The Credit Spread input cell has blue font and a comment referencing actual facility terms.
13. The After-tax Kd cell contains a formula multiplying Pre-tax Kd by (1 - Tax Rate). Verify formula string contains `*(1-` or equivalent.
14. The Wd (Debt Weight) cell contains a formula `=1-` referencing the We cell -- not a hardcoded number. Verify `cell.value` starts with `=`.
15. The WACC cell contains a formula: `We*Re + Wd*Rd(1-T)`. Verify formula references the We, Re, Wd, and After-tax Kd cells.
16. The WACC cell has a double bottom border: `cell.border.bottom.style == "double"`.
17. All formula-computed cells (D/E, Unlev Beta, Re-levered Beta, Re, Kd, WACC) have black font or default font (not blue).
18. All within-Inputs cross-reference cells (Beta in CAPM linking to Re-levered Beta, Rf in Kd linking to Rf in CAPM, Tax in Kd linking to Marginal Tax) have green font: `cell.font.color.rgb` contains `"00FF00"` or equivalent green.

## Known Traps
- **#9: Fabricated peer data** — Every peer Market Cap, Debt, Cash, Beta, Tax must have a real source. No "estimated" values.
- **#15: Peer statistics hardcoded in Python** — AVERAGE and MEDIAN must be Excel formulas that update when peer data changes.
- Wrong Hamada direction (unlevering formula used where re-levering needed, or vice versa)
- Missing CRP in CAPM formula
- Using effective tax rate for peers instead of marginal/statutory
