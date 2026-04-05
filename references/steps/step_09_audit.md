# STEP 9 RED TEAM AUDIT: Depreciation Schedule

You are conducting an ADVERSARIAL audit. Your SOLE PURPOSE is to find errors.

## RULES OF ENGAGEMENT
1. **ASSUME EVERYTHING IS WRONG** until you personally verify each cell is correct
2. **NO BULK PROCESSING** — check each cell INDIVIDUALLY with a targeted command. No loops, no sweeps, no "check all cells in range." ONE CELL AT A TIME.
3. **NO BENEFIT OF THE DOUBT** — if something MIGHT be wrong, it IS wrong. Flag it.
4. **MANUAL COMPUTATION** — for every numerical check, compute the expected value YOURSELF. Do not trust Excel's output without verifying the formula produces it.
5. **WORK EXTRA HARD** — more checks than seem necessary. Redundant verification. If you think you're done, you're not.

## SETUP — Run this FIRST
```bash
pip install openpyxl 2>/dev/null
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
print('Sheets:', wb.sheetnames)
print('Model rows used:', ws.max_row)
print('Model cols used:', ws.max_column)
"
```
If this fails, report BLOCKED immediately.

---

## CHECK 1: Does the Existing Opening formula reference CLOSING? (CATASTROPHIC if wrong)

Read the EXACT formula in the first forecast column's Existing Opening cell:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
cell = ws['{FIRST_FCST}{EX_OPEN_ROW}']
print(f'Cell {cell.coordinate}')
print(f'Value: {cell.value}')
print(f'Type: {type(cell.value)}')
"
```

**EXPECTED**: A formula like `={LAST_HIST}{EX_CLOSE_ROW}` — referencing the CLOSING row of the prior column.
**CATASTROPHIC FAILURE IF**: Formula references `{EX_DEPR_ROW}` (depreciation row) or `{EX_OPEN_ROW}` (itself, circular) or any row OTHER than `{EX_CLOSE_ROW}`.

Extract the row number from the formula. Compare it to the build script's `EX_CLOSE_ROW` constant:
```bash
python3 -c "
import re
formula = '{PASTE_FORMULA_HERE}'
# Extract all row numbers from formula
rows = re.findall(r'[A-Z]+(\d+)', formula)
print(f'Row numbers referenced: {rows}')
# The ONLY row number should match EX_CLOSE_ROW
"
```
Also read the build script to confirm what EX_CLOSE_ROW equals:
```bash
grep -n "EX_CLOSE" "{BUILD_SCRIPT_PATH}" | head -5
grep -n "EX_DEPR" "{BUILD_SCRIPT_PATH}" | head -5
```
**Verify these are DIFFERENT row numbers.** If they're the same or adjacent, this is the #2 catastrophic bug.

---

## CHECK 2: Does depreciation use ACCOUNTING rate, not TAX rate? (CATASTROPHIC if wrong)

Read the depreciation formula:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
cell = ws['{FIRST_FCST}{EX_DEPR_ROW}']
print(f'Cell {cell.coordinate}')
print(f'Formula: {cell.value}')
"
```

**EXPECTED**: Contains `Inputs!$F${ACCT_DEPR_RATE_ROW}` (accounting depreciation rate).
**CATASTROPHIC FAILURE IF**: Contains `Inputs!$F${TAX_RATE_ROW}` or `Inputs!$F${CCA_RATE_ROW}` or `Inputs!$F${USEFUL_LIFE_ROW}`.

Now INDEPENDENTLY verify what value sits in the referenced Inputs cell:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=True)
ws = wb['Inputs']
# Read ALL potentially confusable adjacent rows
for row in range({ACCT_RATE_ROW-2}, {ACCT_RATE_ROW+3}):
    cell = ws[f'F{row}']
    label = ws[f'C{row}'].value
    print(f'  Inputs!F{row} = {cell.value}  (label: {label})')
"
```
**VERIFY**: The referenced value is a RATE between 0.05 and 0.25 (e.g., 0.10 for 10-year life). If it's 0.265 (tax rate) or 0.20-0.30 (CCA rate), the formula is referencing the WRONG row. This is Mistake #1.

---

## CHECK 3: Does new-asset depreciation have HALF-YEAR convention? (STRUCTURAL if missing)

Read the new-asset depreciation formula:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
cell = ws['{FIRST_FCST}{NEW_DEPR_ROW}']
print(f'Formula: {cell.value}')
# Check for 0.5 multiplier
formula = str(cell.value)
has_half_year = '*0.5' in formula or '* 0.5' in formula or '/2' in formula
print(f'Has half-year convention: {has_half_year}')
"
```
**EXPECTED**: Formula contains `*0.5` applied to the current-year capex term.
**STRUCTURAL FAILURE IF**: No `*0.5` or `/2` — means full-year depreciation on new assets in acquisition year. This is Mistake #20.

---

## CHECK 4: Corkscrew algebra — does Closing = Opening + components?

For EACH sub-corkscrew, verify the closing formula:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
# Existing
print('=== EXISTING CORKSCREW ===')
print(f'Opening:  {ws[\"{FIRST_FCST}{EX_OPEN_ROW}\"].value}')
print(f'Depr:     {ws[\"{FIRST_FCST}{EX_DEPR_ROW}\"].value}')
print(f'Closing:  {ws[\"{FIRST_FCST}{EX_CLOSE_ROW}\"].value}')
# New
print('=== NEW ASSET CORKSCREW ===')
print(f'Opening:  {ws[\"{FIRST_FCST}{NEW_OPEN_ROW}\"].value}')
print(f'Capex:    {ws[\"{FIRST_FCST}{NEW_CAPEX_ROW}\"].value}')
print(f'Depr:     {ws[\"{FIRST_FCST}{NEW_DEPR_ROW}\"].value}')
print(f'Closing:  {ws[\"{FIRST_FCST}{NEW_CLOSE_ROW}\"].value}')
"
```
**VERIFY manually**: Existing Closing formula sums Opening + Depr. New Closing formula sums Opening + Capex + Depr. If any component is MISSING from the closing formula, the corkscrew is broken.

---

## CHECK 5: First forecast opening values correct

```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=True)
ws = wb['Model']
ex_open = ws['{FIRST_FCST}{EX_OPEN_ROW}'].value
new_open = ws['{FIRST_FCST}{NEW_OPEN_ROW}'].value
print(f'Existing Opening value: {ex_open}')
print(f'New Opening value: {new_open}')
print(f'New Opening should be 0: {new_open == 0 or new_open is None}')
"
```
**VERIFY**: Existing Opening matches the PP&E NBV from the research file: `{PP&E_NBV_VALUE}`.
**VERIFY**: New Opening = 0 (no new assets in first year's opening).

---

## CHECK 6: MANUAL MATHEMATICAL SPOT-CHECK

Compute depreciation by hand and compare:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=True)
ws_model = wb['Model']
ws_inputs = wb['Inputs']

# Read inputs
opening_nbv = ws_model['{FIRST_FCST}{EX_OPEN_ROW}'].value
acct_rate = ws_inputs['F{ACCT_DEPR_RATE_ROW}'].value
capex = ws_model['{FIRST_FCST}{NEW_CAPEX_ROW}'].value

# Compute expected
expected_ex_depr = -opening_nbv * acct_rate if opening_nbv else 0
expected_new_depr = -(capex * acct_rate * 0.5) if capex else 0
expected_total = expected_ex_depr + expected_new_depr

# Read actual
actual_ex_depr = ws_model['{FIRST_FCST}{EX_DEPR_ROW}'].value
actual_new_depr = ws_model['{FIRST_FCST}{NEW_DEPR_ROW}'].value
actual_total = ws_model['{FIRST_FCST}{DEPR_TOT_ROW}'].value

print(f'Opening NBV: {opening_nbv}')
print(f'Acct Rate: {acct_rate}')
print(f'Capex: {capex}')
print()
print(f'Existing Depr - Expected: {expected_ex_depr:.2f}, Actual: {actual_ex_depr}')
print(f'New Depr - Expected: {expected_new_depr:.2f}, Actual: {actual_new_depr}')
print(f'Total - Expected: {expected_total:.2f}, Actual: {actual_total}')
print()
diff = abs((actual_total or 0) - expected_total)
print(f'DIFFERENCE: {diff:.2f}')
print(f'PASS: {diff < 0.01}')
"
```
**FAIL if difference > $0.01**. This means either the formula or the inputs are wrong.

---

## CHECK 7: Formula consistency across ALL forecast columns

Do NOT loop. Check each column INDIVIDUALLY:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
print('Existing Depr formulas by column:')
for col in ['J', 'K', 'L', 'M', 'N']:
    print(f'  {col}: {ws[f\"{col}{EX_DEPR_ROW}\"].value}')
print()
print('New Depr formulas by column:')
for col in ['J', 'K', 'L', 'M', 'N']:
    print(f'  {col}: {ws[f\"{col}{NEW_DEPR_ROW}\"].value}')
"
```
**VERIFY**: Same formula PATTERN in each column (only column letters change). If ANY column has a different structure, flag it.

---

## CHECK 8: Historical columns are UNTOUCHED (blue values, not formulas)

```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
print('Historical cells (should be values, NOT formulas):')
for row in [{EX_OPEN_ROW}, {EX_DEPR_ROW}, {EX_CLOSE_ROW}]:
    for col in ['F', 'G', 'H', 'I']:
        cell = ws[f'{col}{row}']
        is_formula = isinstance(cell.value, str) and str(cell.value).startswith('=')
        color = cell.font.color.rgb if cell.font.color and cell.font.color.rgb else 'default/theme'
        print(f'  {col}{row}: value={cell.value}, is_formula={is_formula}, font_color={color}')
        if is_formula:
            print(f'    *** FAIL: Historical cell has formula! Should be hardcoded GAAP value ***')
"
```
**FAIL if ANY historical cell (F-I) contains a formula.** Exception: corkscrew opening rows in G-I may reference prior column's closing (this is acceptable for historical corkscrews).

---

## CHECK 9: Font colors correct

```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
print('Font colors for forecast column J:')
for row_name, row_num in [('Ex Open', {EX_OPEN_ROW}), ('Ex Depr', {EX_DEPR_ROW}), ('Ex Close', {EX_CLOSE_ROW}), ('New Open', {NEW_OPEN_ROW}), ('Capex', {NEW_CAPEX_ROW}), ('New Depr', {NEW_DEPR_ROW}), ('New Close', {NEW_CLOSE_ROW}), ('Total Depr', {DEPR_TOT_ROW})]:
    cell = ws[f'J{row_num}']
    if cell.font.color:
        rgb = cell.font.color.rgb
    elif cell.font.color and cell.font.color.theme is not None:
        rgb = f'theme={cell.font.color.theme}'
    else:
        rgb = 'default'
    has_inputs_ref = 'Inputs!' in str(cell.value) if cell.value else False
    expected = 'GREEN (#006100)' if has_inputs_ref else 'BLACK (#000000)'
    print(f'  {row_name} (J{row_num}): rgb={rgb}, expected={expected}')
"
```
**VERIFY**: Formulas referencing Inputs! = green (#006100). Same-sheet formulas = black (#000000). Historical inputs = blue (#0000CC).

---

## CHECK 10: Double border on Total Depreciation row

```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
cell = ws['J{DEPR_TOT_ROW}']
bottom = cell.border.bottom
print(f'Total Depr (J{DEPR_TOT_ROW}):')
print(f'  Bottom border style: {bottom.style}')
print(f'  Expected: double')
print(f'  PASS: {bottom.style == \"double\"}')
"
```

---

## CHECK 11: Total D&A links to IS correctly

```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
# Read the IS D&A formula — should reference this schedule's total
is_da = ws['J{IS_DA_ROW}'].value
da_total = ws['J{DA_TOTAL_ROW}'].value
print(f'IS D&A formula (J{IS_DA_ROW}): {is_da}')
print(f'Depr Schedule Total D&A formula (J{DA_TOTAL_ROW}): {da_total}')
# Verify the IS D&A formula references DA_TOTAL_ROW
print(f'IS references correct row: {\"{DA_TOTAL_ROW}\" in str(is_da)}')
"
```

---

## CHECK 12: Second forecast year — verify accumulation works

Don't just check year 1. Verify year 2 new-asset Opening = year 1 new-asset Closing:
```bash
python3 -c "
import openpyxl
wb = openpyxl.load_workbook('{XLSX_PATH}', data_only=False)
ws = wb['Model']
# Year 2 new opening should reference year 1 new closing
year2_new_open = ws['K{NEW_OPEN_ROW}'].value
print(f'Year 2 New Opening (K{NEW_OPEN_ROW}): {year2_new_open}')
# Should be '=J{NEW_CLOSE_ROW}'
print(f'References New Close: {\"{NEW_CLOSE_ROW}\" in str(year2_new_open)}')
"
```

---

## VERDICT

After completing ALL 12 checks, report:

```
## STEP 9: {PASS | FAILURES}

Checks completed: 12/12
Cells individually inspected: {count each cell you read}
Manual computations: {count spot-checks}
Known traps tested: #1 (Check 2), #2 (Check 1), #20 (Check 3)

{If FAILURES:}
| # | Check | Cell | Severity | Expected | Actual | Issue |
|---|-------|------|----------|----------|--------|-------|

{If PASS:}
All 12 checks passed. Every formula verified individually.
No known traps triggered. Manual computation matched within $0.01.
```
