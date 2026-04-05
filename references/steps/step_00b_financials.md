# Step 0b: Financial Statement Extraction (IS/BS/CF)

## Mandate
Extract ALL income statement, balance sheet, and cash flow statement line items for 4 historical fiscal years.

## Output
- `data/financials.csv` — every IS/BS/CF line item with full provenance
- `data/artifacts/{source_name}/{topic}_{page}.txt` — raw text excerpts

## Required Fields (minimum — extract more if available)

### Income Statement
revenue, cost_of_revenue, gross_profit, sga_expense, other_operating_expense, ebitda, depreciation_amortization, ebit, interest_expense, ebt, income_tax, net_income

### Balance Sheet
cash_equivalents, accounts_receivable, inventory, total_current_assets, ppe_gross, accumulated_depreciation, ppe_net, right_of_use_assets, goodwill, total_assets, accounts_payable, accrued_liabilities, current_debt, long_term_debt, total_liabilities, shareholders_equity

### Cash Flow Statement
operating_cash_flow, capex, free_cash_flow, dividends_paid, shares_repurchased, debt_issued, debt_repaid

## Cross-Check Requirement
For at least 3 values, verify against 2 independent sources. Document both sources in the CSV. If values conflict, flag as `conflict` in a notes column and prefer the higher-priority source.

## Source Priority
1. Annual Report PDF (audited)
2. Press Release (management-prepared)
3. SEDAR/EDGAR filing
4. StockAnalysis / aggregator

## Audit Checklist
- [ ] 1. financials.csv exists with IS/BS/CF sections
- [ ] 2. Every row has source_doc, source_page, and source_quote populated
- [ ] 3. At least 4 fiscal years of data for each major line item
- [ ] 4. At least 3 values cross-checked against 2 independent sources
- [ ] 5. Revenue - COGS = Gross Profit (verify arithmetic for each year)
- [ ] 6. EBITDA = Gross Profit - SGA - Other OpEx (verify)
- [ ] 7. Total Assets = Total Liabilities + Equity (verify for each year)
- [ ] 8. At least 3 artifact files saved with source text
- [ ] 9. All values use consistent units (CAD_M or as specified)
- [ ] 10. Confidence column populated (verified/estimated/derived)
