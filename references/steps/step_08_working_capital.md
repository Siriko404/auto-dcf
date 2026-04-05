# Step 8: Working Capital Schedule (Model Sheet)

AR, Inventory, AP corkscrews with Days-based closing balances. CRITICAL: AR is revenue-driven, but Inventory and AP are COGS-driven.

## Structure
```
Row {hdr}:     "WORKING CAPITAL SCHEDULE" header (bold, bottom border)
Row {blank}:   (blank)

--- Accounts Receivable ---
Row {ar_open}:  AR Opening (= prior year AR Closing)
Row {ar_chg}:   AR Change (= Closing - Opening)
Row {ar_close}: AR Closing

--- Inventory ---
Row {inv_open}:  Inventory Opening (= prior year Inv Closing)
Row {inv_chg}:   Inventory Change (= Closing - Opening)
Row {inv_close}: Inventory Closing

--- Accounts Payable ---
Row {ap_open}:  AP Opening (= prior year AP Closing)
Row {ap_chg}:   AP Change (= Closing - Opening)
Row {ap_close}: AP Closing

--- Summary ---
Row {nwc}:      NWC = AR + Inventory - AP
Row {nwc_pct}:  NWC as % of Revenue
Row {delta}:    Change in NWC = Current NWC - Prior NWC (double border)
```

## Key Formulas

### Corkscrew Pattern (all 3 components)
- Opening (forecast): `={PREV}{CLOSE}` (prior year's Closing)
- Opening (first forecast, col J): `={I}{CLOSE}` (last historical Closing)
- Change: `={COL}{CLOSE}-{COL}{OPEN}`
- Closing: see below (differs by component)

### CRITICAL: Closing Balance Formulas

**AR Closing = Revenue * AR Days / 365** (REVENUE-driven)
```
={COL}${REV_ROW}*Inputs!{COL}${AR_DAYS_ACTIVE}/365
```

**Inventory Closing = COGS * Inv Days / 365** (COGS-driven, NOT Revenue)
```
=-{COL}${COGS_ROW}*Inputs!{COL}${INV_DAYS_ACTIVE}/365
```
Note: If COGS is stored negative, negate it (hence the `-` prefix) so the closing balance is positive.

**AP Closing = COGS * AP Days / 365** (COGS-driven, NOT Revenue)
```
=-{COL}${COGS_ROW}*Inputs!{COL}${AP_DAYS_ACTIVE}/365
```
Same negation logic as Inventory.

### Summary
- NWC: `={COL}{AR_CLOSE}+{COL}{INV_CLOSE}-{COL}{AP_CLOSE}`
- NWC %: `={COL}{NWC}/{COL}${REV_ROW}`
- Change in NWC: `={COL}{NWC}-{PREV}{NWC}`

### Historical (columns F-I)
- AR, Inventory, AP Closing: hardcoded from BS, BLUE font
- Opening: prior year Closing (or hardcoded if first year)
- Change: Closing - Opening

### Historical Corkscrew Population
Historical columns (F-I) MUST be populated with actual BS data (blue font):
- AR Opening/Closing from each year's audited balance sheet
- Inventory Opening/Closing from each year's audited balance sheet
- AP Opening/Closing from each year's audited balance sheet

Do NOT leave historical corkscrew rows empty. The first forecast Opening = last historical Closing must have an actual value to reference.

## Audit Checklist (openpyxl-verifiable)
1. AR Closing = Revenue * AR Days / 365 (Revenue-driven) — verify forecast formula in `ws.cell(AR_CLOSE, col).value` contains `REV_ROW` reference and `/365`, NOT a COGS reference
2. Inventory Closing = COGS * Inv Days / 365 (COGS-driven, Trap #21) — verify forecast formula in `ws.cell(INV_CLOSE, col).value` contains `COGS_ROW` reference and `/365`, NOT a Revenue reference
3. AP Closing = COGS * AP Days / 365 (COGS-driven, Trap #21) — verify forecast formula in `ws.cell(AP_CLOSE, col).value` contains `COGS_ROW` reference and `/365`, NOT a Revenue reference
4. All corkscrews: Opening = Prior Closing — verify `ws.cell(AR_OPEN, col).value` references prior column's `AR_CLOSE` row; same for `INV_OPEN` -> `INV_CLOSE` and `AP_OPEN` -> `AP_CLOSE`; first forecast (col J) references col I
5. Historical opening values from CSV (blue font) — verify `ws.cell(row, col).value` matches CSV for AR/Inv/AP Opening and Closing in columns F-I; `cell.font.color.rgb == '000000CC'`
6. NWC = AR + Inv - AP — verify `ws.cell(NWC_ROW, col).value` contains formula referencing `AR_CLOSE`, `INV_CLOSE`, and `AP_CLOSE` rows with pattern `={COL}{AR_CLOSE}+{COL}{INV_CLOSE}-{COL}{AP_CLOSE}`
7. Delta NWC = Current NWC - Prior NWC — verify `ws.cell(DELTA_NWC, col).value` contains formula `={COL}{NWC}-{PREV}{NWC}` for columns G-N
8. Days display rows link to Inputs ACTIVE — verify `ws.cell(AR_DAYS_DISPLAY, col).value` (and Inv/AP Days display rows) contains `=Inputs!` referencing the ACTIVE driver row; font color is green (`cell.font.color.rgb == '00006100'`)
9. Double border on Delta NWC — verify `ws.cell(DELTA_NWC, col).border.bottom.style == 'double'` for all data columns
10. Source comments on historical values reference CSV — verify `ws.cell(row, col).comment is not None` for historical balance cells (F-I) in AR/Inv/AP Closing rows and comment text contains source reference

## Known Traps
- **#21: Inventory/AP driven by Revenue instead of COGS** — This is the most common WC error. Only AR uses Revenue. Inventory and AP MUST use COGS. When COGS margin shifts, using Revenue for Inv/AP creates material FCF errors.
- **#5: NWC assumption too low** — Validate Days assumptions against actual historical ratios. AR Days = AR/Revenue*365, Inv Days = Inv/COGS*365, AP Days = AP/COGS*365.
- Opening/Closing row confusion (similar to PP&E Mistake #2)
- COGS sign: if COGS is stored negative in the model, the closing formula must negate it to get a positive balance
- Forgetting that the 365 divisor should be consistent (not 360 or 365.25 for WC — use 365)
