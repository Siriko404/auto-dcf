# Step 0e: Market Data

## Mandate
Gather market-level inputs for WACC calculation: risk-free rate, equity risk premium, country risk premium, size premium.

## Output
- `data/market_data.csv` — each rate/premium with full provenance
- `data/artifacts/{source}/` — raw source text

## Required Fields
- risk_free_rate (10-year government bond yield for target's country)
- equity_risk_premium (Damodaran implied ERP or equivalent)
- country_risk_premium (if applicable — 0 for US/Canada mature markets, or Damodaran CRP)
- size_premium (small-cap premium if company qualifies, from Duff & Phelps/Kroll or similar)
- market_return (for reference: Rf + ERP)

## Sources
- Risk-free rate: Central bank website (Bank of Canada, US Treasury)
- ERP: Damodaran's website (pages.stern.nyu.edu/~adamodar/)
- CRP: Damodaran's country risk premium table
- Size premium: Duff & Phelps/Kroll SBBI yearbook or equivalent

## Audit Checklist
- [ ] 1. Risk-free rate sourced from central bank (not estimated)
- [ ] 2. ERP sourced from Damodaran or equivalent academic source
- [ ] 3. CRP appropriate for target's country (0 for mature markets with justification)
- [ ] 4. All values have source_quote with the actual number cited
- [ ] 5. Rates are current (within last 90 days)
- [ ] 6. At least 2 artifact files saved
