# Step 2: Drivers Section (Inputs Sheet)

Three-scenario driver switch with CHOOSE formulas. 8 driver groups with separate AR Days, Inventory Days, and AP Days (not a single NWC%).

## Structure
```
Row {hdr}:    "OPERATING ASSUMPTIONS" section header (bold, bottom border)
Row {blank}:  (blank)
Row {sw_lbl}: "Driver Switch"
Row {switch}: Active Scenario (F7, data validation 1/2/3, default=2)
Row {blank}:  (blank)

--- Revenue Drivers ---
Row {vol_best}:   Volume Growth - Best Case (blue)
Row {vol_base}:   Volume Growth - Base Case (blue)
Row {vol_worst}:  Volume Growth - Worst Case (blue)
Row {vol_active}: Volume Growth - Active = CHOOSE($F$7, best, base, worst)

Row {price_best}:   Pricing Growth - Best (blue)
Row {price_base}:   Pricing Growth - Base (blue)
Row {price_worst}:  Pricing Growth - Worst (blue)
Row {price_active}: Pricing Growth - Active = CHOOSE($F$7, ...)

--- Cost Drivers ---
Row {cogs_best}:   COGS % - Best (blue)
Row {cogs_base}:   COGS % - Base (blue)
Row {cogs_worst}:  COGS % - Worst (blue)
Row {cogs_active}: COGS % - Active = CHOOSE($F$7, ...)

Row {sga_best}:   SGA % - Best (blue)
Row {sga_base}:   SGA % - Base (blue)
Row {sga_worst}:  SGA % - Worst (blue)
Row {sga_active}: SGA % - Active = CHOOSE($F$7, ...)

Row {capex_best}:   Capex % - Best (blue)
Row {capex_base}:   Capex % - Base (blue)
Row {capex_worst}:  Capex % - Worst (blue)
Row {capex_active}: Capex % - Active = CHOOSE($F$7, ...)

--- Working Capital Drivers (3 SEPARATE groups) ---
Row {ar_best}:   AR Days - Best (blue)
Row {ar_base}:   AR Days - Base (blue)
Row {ar_worst}:  AR Days - Worst (blue)
Row {ar_active}: AR Days - Active = CHOOSE($F$7, ...)

Row {inv_best}:   Inventory Days - Best (blue)
Row {inv_base}:   Inventory Days - Base (blue)
Row {inv_worst}:  Inventory Days - Worst (blue)
Row {inv_active}: Inventory Days - Active = CHOOSE($F$7, ...)

Row {ap_best}:   AP Days - Best (blue)
Row {ap_base}:   AP Days - Base (blue)
Row {ap_worst}:  AP Days - Worst (blue)
Row {ap_active}: AP Days - Active = CHOOSE($F$7, ...)
```

## Key Formulas

### Driver Switch Cell (F7)
- Value: `2` (default = Base Case)
- Data Validation: List `1,2,3`

### Active Row Pattern (all 8 groups, all forecast columns J-N)
```
=CHOOSE(Inputs!$F$7, {COL}{BEST_ROW}, {COL}{BASE_ROW}, {COL}{WORST_ROW})
```
- `$F$7` MUST be absolute (dollar signs on both row and column)
- Each forecast column gets its own CHOOSE formula

### NOTE: COGS%/SGA% Alternative
COGS% and SGA% may alternatively be implemented as **per-year direct blue inputs** in the Other Inputs section (rows 116+) rather than CHOOSE-switched scenarios. This is appropriate when:
- Historical COGS%/SGA% varies significantly year to year
- The user wants granular per-year control without scenario switching
- The company has structural margin shifts that don't fit a uniform scenario

If using per-year direct inputs: skip the CHOOSE switch for that group, use blue hardcoded values in the Other Inputs rows, and link the Cost Schedule to those rows instead.

## Audit Checklist (auditor MUST address EVERY item)

1. Cell F7 contains a data validation object (via openpyxl `ws['F7'].data_type` and `ws.data_validations`) with formula1 containing `"1,2,3"` and type `"list"`.
2. Cell F7 value equals `2` (Base Case default).
3. Exactly 8 driver groups exist with labels containing: "Volume Growth", "Pricing Growth", "COGS %", "SGA %", "Capex %", "AR Days", "Inventory Days", "AP Days". Verify by scanning column E for these label substrings.
4. Each driver group has exactly 4 rows: Best, Base, Worst, Active. Verify label text in column E for each group contains the scenario name.
5. Working capital drivers are 3 SEPARATE groups (AR Days, Inventory Days, AP Days) -- confirm 3 distinct Active rows, NOT a single "NWC%" row anywhere.
6. Every Active row cell in forecast columns (J through N) contains a formula starting with `=CHOOSE(` -- read `cell.value` for each and assert it begins with `=CHOOSE(`.
7. Every CHOOSE formula in Active rows contains the substring `$F$7` (fully absolute) -- parse each formula string and confirm both dollar signs present on F and 7.
8. Each CHOOSE formula references exactly the 3 scenario rows for its group (Best, Base, Worst). Verify the row numbers in the formula match the actual row numbers of the Best/Base/Worst cells in that group.
9. All Best/Base/Worst input cells in forecast columns have blue font: `cell.font.color.rgb` equals `"000000CC"` or `"0000CC"` (openpyxl may prefix with alpha).
10. All Active (CHOOSE) cells have black font: `cell.font.color.rgb` equals `"00000000"` or theme-default black, or `cell.font.color` is None (auto-black).
11. Percentage driver cells (COGS%, SGA%, Capex%, Vol Growth, Price Growth) have number format `"0.0%"` -- check `cell.number_format`.
12. Days driver cells (AR Days, Inventory Days, AP Days) have number format `"#,##0"` -- check `cell.number_format`.
13. Every blue-font input cell in forecast columns has a non-None comment: `cell.comment is not None` and `cell.comment.text` contains a source reference (not empty string).
14. Each driver group is separated by at least one blank row from the next group -- verify the row between the Active row of one group and the Best row of the next group has no value in columns E-N.

## Known Traps
- **#5: NWC assumption too low** — Validate AR/Inv/AP Days against actual historical ratios from research file. Do not underestimate.
- **#21: Single NWC%** — Must be 3 separate groups. Inventory and AP Days tie to COGS in Step 8, AR Days ties to Revenue.
- CHOOSE referencing wrong scenario rows (off-by-one when row constants miscounted)
- Missing `$` on F7 reference (breaks when formulas copied across columns)
- Not including data validation on F7 (user can type invalid values)
