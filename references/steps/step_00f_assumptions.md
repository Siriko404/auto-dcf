# Step 0f: Driver Assumptions

## Mandate
Create 3-scenario (Best/Base/Worst) assumptions for all 8 driver groups, justified against historical ratios from financials.csv.

## Output
- `data/assumptions.csv` — each assumption with rationale and historical basis
- No artifacts needed (assumptions are judgment, not sourced data)

## Required Driver Groups (8 total)
1. Volume Growth (%)
2. Pricing Growth (%)
3. COGS % of Revenue
4. SGA % of Revenue
5. Other OpEx % of Revenue
6. Capex % of Revenue
7. AR Days (Revenue-driven)
8. Inventory Days (COGS-driven)
9. AP Days (COGS-driven)

Plus single-value assumptions:
- Terminal Growth Rate (%)
- Exit EBITDA Multiple (x)
- Annual Debt Repayment ($)
- Other D&A per year (ROU + intangible amortization, $)

## CSV Schema (extended for assumptions)
```csv
field,value,unit,scenario,fy,source_doc,source_page,source_quote,retrieval_date,confidence,rationale
vol_growth,0.04,%,best,2026F,,,,,estimated,"Bull: 4% Y1 based on CAGR 3.0% 2022-2025 + market recovery"
vol_growth,0.02,%,base,2026F,,,,,estimated,"Base: 2% Y1, Canadian industrial GDP proxy"
vol_growth,0.00,%,worst,2026F,,,,,estimated,"Bear: 0% Y1, demand contraction scenario"
```

## Requirements
1. Each assumption must reference the historical average from financials.csv
2. Best/Base/Worst must bracket the historical range reasonably
3. Rationale column explains WHY (not just the number)
4. For Days-based drivers: Inventory and AP MUST reference COGS, NOT Revenue (Known Trap #21)

## Audit Checklist
- [ ] 1. All 8 driver groups have Best/Base/Worst values for 5 forecast years
- [ ] 2. Every row has a rationale referencing historical data
- [ ] 3. Base case values are within 1 standard deviation of historical average
- [ ] 4. Best/Worst bracket the historical range (not wildly outside)
- [ ] 5. Terminal Growth Rate < WACC (sanity check)
- [ ] 6. Exit Multiple consistent with peer EV/EBITDA range
- [ ] 7. Single-value assumptions (tax rate, useful life, etc.) all present
