# Step 0g: Data Cross-Validation Gate

## Mandate
Verify ALL data CSVs from sub-steps 0a-0f before any model building begins. This is a BLOCKING gate — the build CANNOT start until this step passes.

## Input Files
- `data/company_profile.csv`
- `data/financials.csv`
- `data/peers.csv`
- `data/market_data.csv`
- `data/assumptions.csv`

## Output
- `{TICKER}_reports/step_00g_validation_report.md` — comprehensive validation report with PASS/FAIL verdict

## Validation Checks

### Arithmetic Consistency
1. Revenue - COGS = Gross Profit (for each FY, tolerance $0.5M)
2. Gross Profit - SGA - Other OpEx = EBITDA (for each FY)
3. EBITDA - D&A = EBIT (for each FY)
4. EBIT - Interest = EBT (for each FY)
5. EBT - Tax = Net Income (for each FY)
6. Total Assets = Total Liabilities + Shareholders' Equity (for each FY)

### PP&E Sanity Check
7. PP&E depreciable net < Total PP&E net (ROU excluded?)
8. PP&E depreciable net is in a reasonable range ($50M-$150M for Wajax-sized company)
9. If ROU assets disclosed: PP&E depreciable / (PP&E depreciable + ROU) < 0.5 would be suspicious

### Historical Ratio Plausibility
10. Gross margin 15-25% (typical industrial distributor)
11. EBITDA margin 5-12%
12. SGA % of revenue 10-20%
13. Capex % of revenue 0.2-2%
14. AR Days 30-70 days
15. Inventory Days 80-180 days
16. AP Days 50-120 days

### WACC Input Verification
17. At least 5 peers in peers.csv
18. All peer beta values between 0.3 and 3.0 (flag outliers)
19. Risk-free rate between 2% and 6% (current macro range)
20. ERP between 4% and 8%
21. Unlevered beta values computed correctly (verify Hamada formula)

### Completeness Check
22. All required fields present (no blanks in required columns)
23. At least 4 fiscal years of IS/BS/CF data
24. All assumption driver groups have Best/Base/Worst
25. No unresolved source conflicts (flag any remaining)

### Source Integrity
26. Every row in financials.csv has source_doc populated
27. Every row with confidence=verified has been cross-checked
28. At least 10 artifact files exist in data/artifacts/

## Verdict Format

```markdown
# Data Validation Report: {TICKER}

## VERDICT: {PASS / FAIL}

### Checks Performed: {count}
### Passed: {count}
### Failed: {count}
### Warnings: {count}

| # | Check | Result | Details |
|---|-------|--------|---------|
| 1 | Revenue - COGS = GP | PASS | FY2022: 1962.8 - 1573.0 = 389.8 (matches GP 389.8) |
| 2 | ... | ... | ... |

### Failures (must fix before build)
{Detailed description of each failure with suggested fix}

### Warnings (review but not blocking)
{Issues that are suspicious but not definitively wrong}
```

## Audit Checklist
- [ ] 1. All 27 validation checks performed
- [ ] 2. Arithmetic checks pass within tolerance
- [ ] 3. PP&E excludes ROU (check #7-9 pass)
- [ ] 4. Historical ratios within plausible ranges
- [ ] 5. WACC inputs present and reasonable
- [ ] 6. No unresolved source conflicts
- [ ] 7. Completeness check passes (no missing required fields)
- [ ] 8. Final verdict clearly stated as PASS or FAIL
