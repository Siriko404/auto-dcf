# Step 0a: Company Identification

## Mandate
Identify the target company and document basic profile information.

## Output
- `data/company_profile.csv` with fields: name, ticker, exchange, currency, fy_end, auditor, industry, hq_address, shares_outstanding_diluted
- `data/artifacts/{source}/company_profile_{page}.txt` for each source consulted

## Process
1. Search for the company's investor relations page
2. Confirm ticker and exchange (TSX, NYSE, NASDAQ, etc.)
3. Identify fiscal year end date
4. Identify external auditor
5. Get latest diluted shares outstanding
6. Save all source text as artifacts

## CSV Schema
```csv
field,value,unit,fy,source_doc,source_page,source_quote,retrieval_date,confidence,artifact_file
company_name,Wajax Corporation,text,,Annual Report 2024,Cover,"Wajax Corporation",2026-04-02,verified,artifacts/2024_AR/cover.txt
ticker,WJX,text,,TSX,,TSX: WJX,2026-04-02,verified,
exchange,TSX,text,,,,,,verified,
currency,CAD,text,,Annual Report 2024,p.1,"amounts in Canadian dollars",2026-04-02,verified,artifacts/2024_AR/cover.txt
```

## Audit Checklist
- [ ] 1. company_profile.csv exists with all required fields
- [ ] 2. Every row has source_doc and source_page populated
- [ ] 3. At least 1 artifact file saved
- [ ] 4. Ticker confirmed against actual exchange listing
