# Step 5: Revenue Schedule (Model Sheet)

First schedule in Model. Historical GAAP revenue (hardcoded) plus forecast revenue linked to volume and pricing drivers from Inputs.

## Structure
```
Row {hdr}:    "REVENUE SCHEDULE" header (bold, bottom border)
Row {blank}:  (blank)
Row {rev}:    Revenue
Row {vol}:    Volume Growth % (green -> Inputs active driver)
Row {price}:  Pricing Growth % (green -> Inputs active driver)
Row {growth}: Revenue Growth YoY %
```

## Key Formulas

### Historical (columns F-I)
- Revenue: hardcoded GAAP value, BLUE font, source comment
- Vol/Price: blank or historical computed
- YoY Growth: `={COL}{REV}/{PREV}{REV}-1`

### Forecast (columns J-N)
- Revenue: `={PREV}{REV}*(1+Inputs!{COL}${VOL_ACTIVE})*(1+Inputs!{COL}${PRICE_ACTIVE})`
  - Links to ACTIVE driver rows (CHOOSE output), NOT scenario rows
  - `$` on Inputs row references (absolute row)
  - First forecast (J) references last historical (I)
- Vol Growth display: `=Inputs!{COL}${VOL_ACTIVE}` (GREEN font)
- Price Growth display: `=Inputs!{COL}${PRICE_ACTIVE}` (GREEN font)
- YoY Growth: `={COL}{REV}/{PREV}{REV}-1` (BLACK font)

## Audit Checklist (openpyxl-verifiable)
1. Historical revenue (F-I) matches CSV values exactly — verify `ws.cell(REV_ROW, col).value == expected_value` for columns F through I against the research CSV; font color is blue (`cell.font.color.rgb == '000000CC'`)
2. Forecast revenue = Prior * (1+Vol) * (1+Price), referencing ACTIVE driver rows — verify `ws.cell(REV_ROW, col).value` starts with `=` and contains `Inputs!` references to `VOL_ACTIVE` and `PRICE_ACTIVE` row numbers (not scenario rows); formula pattern: `={PREV}{REV}*(1+Inputs!{COL}${VOL_ACTIVE})*(1+Inputs!{COL}${PRICE_ACTIVE})`
3. Vol/Price display rows link to Inputs ACTIVE rows — verify `ws.cell(VOL_GROWTH, col).value` contains `=Inputs!` and references the ACTIVE driver row; font color is green (`cell.font.color.rgb == '00006100'`)
4. YoY growth computed for all columns — verify `ws.cell(REV_YOY_GROWTH, col).value` contains a formula like `={COL}{REV}/{PREV}{REV}-1` for every column G through N (F may be blank or hardcoded)
5. Source comments reference CSV row — verify `ws.cell(REV_ROW, col).comment is not None` for historical columns F-I and comment text contains a source reference (e.g., "IS Row" or CSV line number)
6. Font colors: blue historical, green cross-sheet, black formulas — verify: historical data cells (F-I) have `font.color.rgb == '000000CC'`; cross-sheet formula cells have `font.color.rgb == '00006100'`; computed formula cells (YoY growth) have `font.color.rgb == '00000000'` or theme=1

## Known Traps
- Linking to a scenario row (Best/Base/Worst) instead of the CHOOSE Active row
- Missing `$` on Inputs row reference (row reference shifts when formula copied across columns)
- First forecast column referencing wrong historical column
- Using "adjusted" revenue instead of GAAP (#7 — unlikely for revenue, but verify)
