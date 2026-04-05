# Step 0c: Note Disclosures Deep Dive

## Mandate
Extract detailed data from annual report notes that isn't in the summary financial statements. Focus on PP&E breakdown (CRITICAL: exclude ROU), tax rate details, debt terms, and share information.

## Output
- Additional rows appended to `data/financials.csv`
- `data/artifacts/{AR_year}/note_{number}_{topic}.txt` — raw text of each relevant note

## CRITICAL: PP&E Definition
PP&E for the depreciation schedule = Property/Plant/Equipment + Rental Equipment ONLY.
**EXCLUDE Right-of-Use assets (IFRS 16 leases).** In prior builds, including ROU inflated PP&E from $85M to $235M, overstating depreciation 2-3x.

You MUST find the note disclosure that breaks down total PP&E into:
- Owned property, plant and equipment
- Rental equipment
- Right-of-use assets (EXCLUDE this)
- Other (vehicles, IT, etc.)

Save the full note text as an artifact. The source_quote in CSV must explicitly show the breakdown.

## Required Note Extractions

### PP&E Breakdown (Note 9 or similar)
- ppe_owned_gross, ppe_owned_net, rental_equipment_gross, rental_equipment_net
- ppe_depreciable_net (= owned + rental, EXCLUDING ROU)
- right_of_use_assets_net (for documentation only — NOT used in model)
- useful_life_buildings, useful_life_equipment, useful_life_rental

### Tax (Note 24 or similar)
- statutory_tax_rate (combined federal + provincial, NOT effective rate)
- current_tax_expense, deferred_tax_expense (if disclosed)

### Debt (Note 17 or similar)
- bank_debt, debentures, lease_liabilities (separate each)
- interest_rate_range, maturity_dates
- funded_debt = bank_debt + debentures (EXCLUDE lease liabilities)

### Shares (Note 19 or similar)
- shares_diluted_weighted_avg (for each FY)

## Audit Checklist
- [ ] 1. PP&E breakdown found and ROU EXCLUDED from depreciable base
- [ ] 2. source_quote for PP&E explicitly shows the breakdown (not just a total)
- [ ] 3. Statutory tax rate extracted (NOT effective rate)
- [ ] 4. Funded debt excludes lease liabilities
- [ ] 5. At least 4 artifact files saved (one per major note)
- [ ] 6. All new rows in financials.csv have full provenance
- [ ] 7. PP&E depreciable net is significantly LESS than total PP&E (sanity check)
