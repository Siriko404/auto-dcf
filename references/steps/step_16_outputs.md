# Step 16: Outputs Dashboard (Outputs Sheet)

Two dashboards (Perpetuity + Multiple) with sensitivity tables and scenario comparison. Sensitivity formulas are 300+ characters and must be built programmatically.

## Structure (repeat for BOTH Perpetuity and Multiple dashboards)
```
--- Dashboard Header ---
Row {title}:    Dashboard Title ("Perpetuity Growth Method" / "EBITDA Multiple Method")
Row {scenario}: Active Scenario = CHOOSE(Inputs!$F$7, "Best Case", "Base Case", "Worst Case")
Row {blank}:    (blank)

--- Key Metrics ---
Row {wacc_lbl}: WACC (green -> Inputs)
Row {tgr_lbl}:  TGR or Exit Multiple (green -> Inputs)
Row {ev_lbl}:   Enterprise Value (green -> Model DCF)
Row {price_lbl}: Implied Share Price (green -> Model DCF)

--- EV Bridge ---
Row {pv_fcf}:   Sum of PV FCFs (green -> Model)
Row {pv_tv}:    PV of Terminal Value (green -> Model)
Row {ev}:       Enterprise Value (green -> Model)
Row {debt}:     Less: Debt (green -> Model)
Row {cash}:     Plus: Cash (green -> Model)
Row {equity}:   Equity Value (green -> Model)
Row {shares}:   Shares (green -> Model)
Row {price}:    Per Share (green -> Model)

--- Sensitivity Table 1: WACC vs TGR (or WACC vs Multiple) ---
{5x5 or 7x7 grid, ODD dimensions}
Row axis: WACC increments (e.g., -2%, -1%, base, +1%, +2%)
Col axis: TGR or Multiple increments
Center cell = base case (highlighted)

--- Sensitivity Table 2: WACC vs Revenue Growth ---
--- Sensitivity Table 3: WACC vs EBITDA Margin ---
--- Sensitivity Table 4: TGR vs EBITDA Margin (or Exit Multiple vs EBITDA Margin) ---
{Same structure for all tables}

--- Scenario Comparison ---
Row {best_price}:  Best Case per share (cumulative compound)
Row {base_price}:  Base Case per share (green -> Model)
Row {worst_price}: Worst Case per share (cumulative compound)
```

### Required Sensitivity Tables (4 per dashboard, 8 total)
Each dashboard (Perpetuity + Multiple) MUST have:
1. WACC vs Terminal Growth Rate (or WACC vs Exit Multiple)
2. WACC vs Revenue Growth
3. WACC vs EBITDA Margin
4. Terminal Growth Rate vs EBITDA Margin (or Exit Multiple vs EBITDA Margin)

Do NOT build only 2 tables. The first live test produced only 2 of 8 required tables.

## CRITICAL: Sensitivity Formula Construction

Each sensitivity cell formula is **300+ characters** and MUST be built programmatically in Python using string concatenation. These formulas re-discount EVERY FCF at the scenario WACC:

```python
# Python pattern for building each sensitivity cell formula
def build_sensitivity_formula(wacc_cell, tgr_cell, fcst_cols, ufcf_row, period_row,
                               tv_row, tv_period_row, debt_ref, cash_ref, shares_ref):
    terms = []
    for col in fcst_cols:
        terms.append(f"Model!{col}${ufcf_row}/(1+{wacc_cell})^Model!{col}${period_row}")
    
    # Terminal value (use Gordon Growth for perpetuity, EBITDA*mult for multiple)
    tv_term = f"Model!{fcst_cols[-1]}${tv_row}/(1+{wacc_cell})^Model!{fcst_cols[-1]}${tv_period_row}"
    
    formula = f"=({'+'.join(terms)}+{tv_term}-{debt_ref}+{cash_ref})/{shares_ref}"
    return formula
```

**Each cell re-discounts ALL FCFs at the new WACC.** This is NOT linear scaling.

### What linear scaling looks like (WRONG):
```
=base_EV * (base_WACC / new_WACC)   <-- WRONG, linear approximation
=base_price * some_ratio             <-- WRONG
```

### What correct re-discounting looks like:
```
=(Model!J$150/(1+$B5)^Model!J$148 + Model!K$150/(1+$B5)^Model!K$148 + ... 
  + Model!N$200/(1+$B5)^Model!N$198 - Model!N$180 + Inputs!$F$104) / Inputs!$F$118
```

## CRITICAL: Scenario Comparison (Cumulative Compound)

### Scenario Comparison (REQUIRED -- not optional)
Each dashboard MUST include Best/Base/Worst per-share values using CUMULATIVE compound differentials.
This was completely skipped in the first live test. It is a institutional requirement (SKILL.md Rule #10).

Best/Worst scenario comparisons use **cumulative compound differentials**, NOT single-year scaling:

```
For year t: factor(t) = PRODUCT(k=1..t, (1 + g_scenario(k)) / (1 + g_base(k)))
Scenario_FCF(t) = Base_FCF(t) * factor(t)
```

Formula pattern for scenario per-share at year t:
  cumulative_factor(t) = PRODUCT(k=1..t, (1+g_scenario(k))/(1+g_base(k)))
  Scenario FCF(t) = Base FCF(t) * cumulative_factor(t)
Then re-discount all scenario FCFs at base WACC to get scenario EV and per-share.

**Wrong (single-year):** `Base_FCF * (1 + bear_g) / (1 + base_g)` per year
**Right (cumulative):** Uses PRODUCT across all years through year t

## Audit Checklist
1. TWO dashboards present (Perpetuity + Multiple) — scan the Outputs sheet for both dashboard title rows; confirm one contains "Perpetuity" and the other contains "Multiple" in cell values
2. 4 sensitivity tables per dashboard = 8 TOTAL (WACC vs TGR, WACC vs Revenue Growth, WACC vs EBITDA Margin, TGR/Multiple vs EBITDA Margin) — count distinct sensitivity table header rows on the Outputs sheet and confirm exactly 8; verify axis labels match these four pairings per dashboard
3. Each sensitivity cell re-discounts ALL FCFs individually (Trap #4, no linear scaling) — read at least 3 non-center sensitivity cell formulas and confirm each contains individual FCF/(1+WACC)^period terms for every forecast year; reject any formula using ratio/proportion scaling
4. Scenario label via CHOOSE (text, not number) — read the scenario cell formula and confirm it contains `CHOOSE(Inputs!$F$XX,"Best Case","Base Case","Worst Case")` returning text strings, not a numeric index (Trap #14)
5. Cumulative scenario comparison present (Best/Base/Worst per share) — locate the scenario comparison section on each dashboard; confirm Best, Base, and Worst per-share rows exist with non-empty values
6. Center cells match base case values — for each sensitivity table, read the center cell value and the base case implied share price from the EV bridge; confirm they match within $0.01
7. All Outputs formulas use Model! or Inputs! sheet prefix — read a sample of 10+ cell formulas from the Outputs sheet and confirm every cross-sheet reference includes the explicit sheet prefix (Trap #11); no bare cell references to other sheets
8. Double borders on implied share price rows — for each dashboard's implied share price row, read border.bottom.style for every cell and confirm "double"
9. Scenario comparison uses cumulative compound differentials (Trap #3) — read the Best/Worst scenario per-share cell formulas and confirm they use PRODUCT across all forecast years with `(1+g_scenario)/(1+g_base)` factors; reject single-year ratio formulas
10. At least 85 Outputs rows (matching reference model scope) — read the Outputs sheet max_row and confirm it is >= 85; insufficient rows indicates missing tables or sections

## Known Traps
- **#4: Linear WACC scaling** -- Must re-discount each FCF. A simple ratio `PV * (WACC_base/WACC_new)` is wrong because discounting is exponential, not linear.
- **#3: Single-year scenario scaling** -- Cumulative product required. Single-year ratio underestimates divergence in later years.
- **#14: Scenario label shows "2" instead of "Base Case"** -- CHOOSE formula must return text strings.
- **#11: Missing sheet prefix** -- Every formula on Outputs must include `Model!` or `Inputs!` prefix.
- **#15: Hardcoded statistics** -- Any computed values must use Excel formulas.
