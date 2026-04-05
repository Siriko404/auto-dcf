# WACC Methodology Reference

## Capital Asset Pricing Model (CAPM)

### Cost of Equity
```
Re = Rf + CRP + ERP × β_L + Size Premium
```

| Component | Description | Source |
|-----------|-------------|--------|
| Rf | Risk-free rate | 10-year government bond yield (matching currency) |
| CRP | Country Risk Premium | Damodaran country risk data (0% for US/Canada core) |
| ERP | Equity Risk Premium | Damodaran or market consensus (~5.5-6.5%) |
| β_L | Re-levered Beta | Hamada equation output (see below) |
| Size | Size Premium | Duff & Phelps (optional, 0 if not used) |

**Key**: CRP is ALWAYS included in the formula, even if 0. This ensures the model is portable to non-US companies.

## Hamada Beta Equations

### Unlevering (from observed peer data)
```
β_U = β_L / (1 + (1 - T) × D/E)
```
Removes the effect of each peer's unique capital structure to isolate business risk.

### Re-levering (to target capital structure)
```
β_L = β_U × (1 + (1 - T) × D/E_target)
```
Applies the subject company's target D/E to the pure business risk beta.

### Peer Data Required (per company)
| Field | Source |
|-------|--------|
| Market Capitalization | Market data (real-time or recent) |
| Total Debt | Most recent balance sheet (include ST + LT debt) |
| Cash & Equivalents | Most recent balance sheet |
| Levered Beta | Market data provider (2Y weekly or 5Y monthly) |
| Tax Rate | Statutory marginal rate for jurisdiction |

### Computations
```
D/E = Total Debt / Market Cap  (or Net Debt / Equity)
Unlevered Beta = Levered Beta / (1 + (1-Tax) × D/E)
```

### Selecting the Unlevered Beta
Compute BOTH:
- **Average** unlevered beta across 5 peers: `=AVERAGE(range)` in Excel
- **Median** unlevered beta across 5 peers: `=MEDIAN(range)` in Excel

Use Excel formulas, NEVER hardcode the statistics. The selected beta is typically the median (more robust to outliers), but present both for transparency.

## Cost of Debt

### Pre-tax Cost of Debt
```
Kd = Rf + Credit Spread
```

| Source for Credit Spread | Priority |
|--------------------------|----------|
| Actual facility terms from filing | 1st (e.g., "CORRA + 175-330 bps") |
| Credit rating → synthetic spread | 2nd |
| Comparable company spreads | 3rd |

### After-tax Cost of Debt
```
Kd(1-T) = Kd × (1 - Marginal Tax Rate)
```
Use MARGINAL (statutory) tax rate, not effective rate. The tax shield is based on the marginal benefit of the next dollar of interest.

## WACC Formula

```
WACC = We × Re + Wd × Kd(1-T)
```

| Weight | Definition |
|--------|-----------|
| We | Equity / (Equity + Debt) — target capital structure |
| Wd | Debt / (Equity + Debt) = 1 - We |

### Current vs Target Capital Structure
- **Current**: Use actual market cap and total debt from most recent BS
- **Target**: Use management guidance, peer median, or analyst consensus
- institutional models typically use a target structure (blue input cell)

## Dual Tax Method

### Why Two Tax Schedules?

UFCF represents cash flows available to ALL capital providers (debt + equity). The tax calculation must reflect the tax an all-equity firm would pay — this is the **unlevered tax**.

### Levered Tax (from EBT)
```
Levered Tax = max(0, EBT) × Statutory Rate
```
This is the actual tax the company pays, AFTER the interest deduction.

### Unlevered Tax (from EBIT)
```
Unlevered Tax = max(0, EBIT) × Statutory Rate
```
This is the hypothetical tax an all-equity firm would pay — NO interest deduction.

### Tax Shield
```
Tax Shield = Unlevered Tax - Levered Tax
```
Or equivalently: `Tax Shield = Interest Expense × Tax Rate`

The tax shield is the tax benefit of debt. It's captured in the WACC (through Kd(1-T)), so the UFCF uses the UNLEVERED tax to avoid double-counting.

### In UFCF
- **EBITDA Method**: UFCF = EBITDA - **Unlevered Tax** - Capex - ΔNWC
- **NI Method**: UFCF = NI + D&A + Interest×(1-T) - Capex - ΔNWC

Both methods should produce the same UFCF (reconciliation = 0).

## Terminal Value

### Gordon Growth (Perpetuity Method)
```
TV = FCF_last × (1 + g) / (WACC - g)
```
Where:
- FCF_last = last forecast year UFCF
- g = terminal growth rate (should be ≤ long-term GDP growth, typically 2-3%)
- WACC - g must be > 0 (model check #3)

### EBITDA Exit Multiple
```
TV = EBITDA_last × Exit Multiple
```
Where:
- EBITDA_last = last forecast year EBITDA
- Exit Multiple from comparable transactions or trading comps

### Discounting Convention
- Discrete FCFs: **mid-year** convention (cash flows assumed received at mid-point)
  - Period = 0.5, 1.5, 2.5, 3.5, 4.5 (for 5-year forecast)
- Terminal Value: **end-of-year** convention (perpetuity starts at end of last year)
  - Period = 5.0 (integer, NOT 4.5)

### Discount Factor
```
DF = 1 / (1 + WACC)^period
```

### Present Value
```
PV of FCF = FCF × DF
PV of TV = TV × DF_terminal
```

## Enterprise Value Bridge

```
Enterprise Value = Sum of PV(FCFs) + PV(TV)
Equity Value = EV - Total Debt + Cash
Per Share = Equity Value / Shares Outstanding
```

All bridge components must reference input cells — no hardcoded numbers in the bridge.
