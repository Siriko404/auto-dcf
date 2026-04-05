> **DEPRECATED in v3.** Data acquisition is now handled by sub-steps 0a through 0g. See step_00a_company_id.md through step_00g_validation.md.

# Step 0: Research

Gather every financial input from actual SEC/SEDAR filings and market data sources. Every number in the model must trace to a real source document.

## Deliverable
`{TICKER}_Research.md` — a structured markdown file with every data point sourced.

## Required Data Sections

### Company Overview
- Ticker, exchange, fiscal year-end
- Business description (1-2 sentences)
- Reporting currency

### Historical Income Statement (4 years)
```
Revenue, COGS, Gross Profit, SGA, Other Opex, EBITDA, D&A, EBIT,
Interest Expense, EBT, Tax Expense, Net Income
Source: Annual Report / 10-K / SEDAR filing with page/note reference
```

### Historical Balance Sheet (4 years)
```
Accounts Receivable, Inventory, Accounts Payable,
PP&E (Net), Total Debt, Cash & Equivalents,
Shares Outstanding (basic + diluted)
Source: Annual Report / 10-K with page reference
```

**CRITICAL: PP&E Definition**
PP&E for depreciation purposes = Property/Plant/Equipment + Rental Equipment ONLY.
EXCLUDE Right-of-Use assets (IFRS 16 / ASC 842 leases) — these are lease obligations, not depreciable capex PP&E.
If the balance sheet shows a combined "Property, plant and equipment" line that includes ROU assets, break it down from the notes.
Failure to exclude ROU assets inflates the depreciable base by 2-3x (e.g., $235M total vs $85M PP&E-only for Wajax).

### WACC Inputs
```
5 comparable companies: Market Cap, Total Debt, Cash, Levered Beta, Tax Rate
Source: Bloomberg/Capital IQ/Yahoo Finance with access date
Risk-Free Rate: 10-year government bond yield (source + date)
ERP: Damodaran or Duff & Phelps (source + edition)
CRP: Damodaran by country (if applicable)
Credit Spread: From actual credit facility terms or rating
```

### Other Inputs
```
Statutory Tax Rate: From tax notes in filing (NOT effective rate)
PP&E Useful Life: From accounting policy notes
CCA/Tax Depreciation Rate: From tax notes
Debt Interest Rate: From credit facility disclosure
```

- **Debt Details**: Facility type, Rate, Spread, Maturity, Mandatory repayment schedule, Covenants

### Driver Assumptions
```
Volume Growth, Pricing Growth: From MD&A, industry data
COGS%, SGA%: Historical ratios + management guidance
Capex%: Historical + guidance
AR Days, Inventory Days, AP Days: Computed from historical BS/IS
Terminal Growth Rate: GDP proxy or industry consensus
Exit EBITDA Multiple: Comparable transactions / trading comps
```

## Key Formulas for Derived Inputs
- AR Days = AR / Revenue * 365
- Inventory Days = Inventory / COGS * 365
- AP Days = AP / COGS * 365
- COGS% = COGS / Revenue
- SGA% = SGA / Revenue
- Capex% = Capex / Revenue (use cash flow statement capex)

## Acceptance Criteria
- [ ] Every number has a named source (filing name, page, note number, or URL)
- [ ] Historical IS covers 4 fiscal years
- [ ] Historical BS covers 4 fiscal years (same periods)
- [ ] GAAP figures used (not "adjusted" EBITDA — see Known Trap #7)
- [ ] 5 real peer companies identified with sourced market data
- [ ] Risk-free rate, ERP, CRP all sourced
- [ ] Tax rate is STATUTORY (from tax notes), not effective
- [ ] AR/Inv/AP Days computed correctly (Inv and AP use COGS, not Revenue)
- [ ] PP&E useful life is in YEARS (not a percentage)
- [ ] Accounting depreciation rate = 1 / useful life (or company-specific if disclosed)
- [ ] Tax/CCA depreciation rate sourced from tax notes
- [ ] Research file saved as `{TICKER}_Research.md`

## Known Traps
- **#7: "Adjusted" EBITDA** — Use GAAP IS figures only. Management-adjusted numbers exclude real costs.
- **#8: Wrong PP&E value** — Must match audited BS exactly. Do not estimate.
- **#9: Fabricated peer data** — Every peer data point must have a source. No "estimated" values.
- **#17: Numbers not traceable** — Every blue-font cell in the model must trace back to this file.
- **#21: Inventory/AP driven by Revenue** — Compute Inv Days and AP Days using COGS, not Revenue.
