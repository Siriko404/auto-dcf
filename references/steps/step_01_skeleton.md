# Step 1: Workbook Skeleton

## What to Build
Create the 4-tab workbook with exact reference template structure: banner rows, navigation, year headers, schedule headers, column widths, freeze panes, images, footers, print areas, and define ALL row constants in `row_map.py`.

**Source of truth:** Financial Modeling Guidelines (REQ-G) take precedence for conventions. Excel extraction (`extract_complete_excel_SCRIPT.md`) provides exact structure/layout. Where they conflict, GUIDELINES WIN — the template is one implementation that may deviate from the published standard.

## Python Architecture

```python
import openpyxl
from openpyxl.styles import Font, Border, Side, Alignment, PatternFill, Protection
from openpyxl.drawing.image import Image
from openpyxl.utils import get_column_letter

wb = openpyxl.Workbook()

# === STYLE CONSTANTS ===
FONT_DEFAULT = Font(name='Open Sans', size=10)
FONT_BLUE_INPUT = Font(name='Open Sans', size=10, color='FF3271D2')  # hardcoded inputs (REQ-G-108)
FONT_BLUE_INPUT_BOLD = Font(name='Open Sans', size=10, bold=True, color='FF3271D2')
FONT_GREEN_XREF = Font(name='Open Sans', size=10, color='FF006100')  # cross-tab links (REQ-G-110)
FONT_GREEN_XREF_BOLD = Font(name='Open Sans', size=10, bold=True, color='FF006100')
FONT_BLACK_FORMULA = Font(name='Open Sans', size=10, color='FF000000')  # same-sheet formulas (REQ-G-109)
FONT_HEADER = Font(name='Open Sans', size=14, bold=True, color='FF3271D2')  # schedule headers
FONT_HEADER_PURE_BLUE = Font(name='Open Sans', size=14, bold=True, color='FF0000FF')  # Inputs sub-header B4
FONT_WHITE = Font(name='Open Sans', size=10, bold=True, color='FFFFFFFF')  # on dark fills
FONT_ORANGE_NAV_BOLD = Font(name='Open Sans', size=11, bold=True, color='FFFA621C')  # row 2 B-C (no italic)
FONT_ORANGE_NAV_BI = Font(name='Open Sans', size=11, bold=True, italic=True, color='FFFA621C')  # row 2 D-N
FONT_SCENARIO = Font(name='Open Sans', size=9, italic=True)  # row 5-6 info text
FONT_YEAR = Font(name='Open Sans', size=10, bold=True)  # year headers

FILL_BANNER = PatternFill(start_color='FF000C3F', end_color='FF000C3F', fill_type='solid')  # row 1
FILL_SECTION = PatternFill(start_color='FFE7F2FF', end_color='FFE7F2FF', fill_type='solid')  # schedule headers

BORDER_YEAR_HIST = Border(bottom=Side(style='medium', color='FF3271D2'))
BORDER_YEAR_FCST = Border(top=Side(style='thin', color='FF3271D2'), bottom=Side(style='medium', color='FF3271D2'))
BORDER_HAIR = Border(
    left=Side(style='hair', color='FF000000'), right=Side(style='hair', color='FF000000'),
    top=Side(style='hair', color='FF000000'), bottom=Side(style='hair', color='FF000000'))

# === COLUMN DEFINITIONS ===
HIST_COLS = ['F', 'G', 'H']       # 3 historical years (format 0"A")
FCST_COLS = ['I', 'J', 'K', 'L', 'M']  # 5 forecast years (format 0"F")
TERM_COL = 'N'                     # Terminal year (format 0"F", label "Term")
DATA_COLS = HIST_COLS + FCST_COLS + [TERM_COL]  # F through N
```

## Font Color Convention (REQ-G-108 through REQ-G-111 — MANDATORY)

| Cell Type | Font Color | Hex |
|-----------|-----------|-----|
| Hardcoded inputs (user-entered) | **Blue** | `FF3271D2` |
| Formulas / same-sheet links | **Black** | `FF000000` or no explicit color |
| Cross-sheet links (inter-tab) | **Green** | `FF006100` |

**The template omits green — we implement the FULL guideline standard.**

Builder agents MUST apply green font to ANY formula that references a different sheet (e.g., `=Inputs!$F$97` on Model sheet gets green font, not black). This is the published convention (REQ-G-110) even though their own template implementation skipped it.

## Capitalization Convention (REQ-G-123 through REQ-G-127 — MANDATORY)

| Element | Case | Example |
|---------|------|---------|
| Schedule titles | Title Case | Revenue Schedule, Income Statement |
| Section headers within schedules | **UPPER CASE + bold** | OPERATIONS, VOLUME, PRICING, REVENUE |
| Row labels | Title Case (consistent throughout) | Sales Volume Growth, Unit Price |

**REQ-G-125:** Use UPPER CASE and bold to clearly define major sections within a schedule.

## Tab Structure & Order
1. **Cover** (leftmost) — presentation, checks, ToC
2. **Outputs** — dual-panel dashboard (Perpetuity + Multiple methods)
3. **Inputs** — Drivers, WACC, Other Inputs sections
4. **Model** — 11 stacked schedules

## Per-Sheet Column Widths

**Cover** (exact extraction values):
A=4.7109375, B=4.85546875, C=36.7109375, D-K=10.7109375, L=26.7109375, M=10.7109375, N=4.85546875

**Outputs** (dual-panel):
A=9.140625, B=2.7109375, C=8.0, D-H=11.7109375, I=6.7109375, J=2.7109375, K=8.0, L-P=11.7109375, Q=3.7109375

**Inputs:**
A=9.140625, B=12.85546875, C-E=11.85546875, F=12.28515625, G-L=11.85546875, M=1.7109375

**Model:**
A=9.140625, B=17.7109375, C=11.28515625, D=15.7109375, E=1.7109375, F=11.7109375, G-N=10.28515625, O=1.7109375

## Row 1: Banner Row (Outputs, Inputs, Model — NOT Cover row 1)
- Height: 50.1 (Outputs, Inputs, Model)
- Fill: `FF000C3F` (dark navy) on cols B through last data col
- Cols E+ on non-Cover sheets: bg variant `FF000000`
- company logo image: Outputs/Inputs/Model from=(1,0); size 16200x4708 EMU
- No text content — purely visual banner

**Cover is DIFFERENT:** Cover row 1 has white fill (theme=0) on A1 only. The Cover banner fill (`FF000C3F`) is on rows 2-9 (cols B-N). Cover image is at from=(2,3). Cover does NOT follow the same row 1 pattern.

## Row 2: Navigation Row (Model only)
- B2: 11pt **bold** (NOT italic), `FFFA621C` orange, h=center
- C2: 11pt **bold** (NOT italic), `FFFA621C` orange, NO alignment
- D2-E2: 11pt bold+italic, `FFFA621C` orange, NO alignment
- F2-N2: 11pt bold+italic, `FFFA621C` orange, h=center
- All: UNLOCKED cells, Comma 2 named style
- Empty values — formatted placeholder for navigation labels

## Model Rows 3-6: Schedule Header Block

### Row 3: First Schedule Header
- B3: `'Revenue Schedule'` — 14pt bold, `FF3271D2` font, `FFE7F2FF` fill, v=center
- C3: 10pt **bold**, `theme=0` (white) font, `FFE7F2FF` fill
- D3-E3: 10pt regular (NOT bold), `theme=0` font, `FFE7F2FF` fill
- F3-M3: 10pt bold, `theme=0` font, `FFE7F2FF` fill, format `0"A"`, h=right
- N3: 10pt bold+italic, `FF3271D2` font, `FFE7F2FF` fill, h=right, v=center

### Row 4: Sub-header (collapse indicator)
- B4: 14pt bold `FF3271D2` font, NO fill (unlike row 3)
- C4-N4: same as row 3 but NO fill

### Row 5: Year Headers
- B5: `'All figures in USD thousands unless stated'` — 9pt italic
- F5: `=G5-1` → 2020, format `0"A"`, border: bottom medium brand blue
- G5: `=H5-1` → 2021, format `0"A"`, border: bottom medium brand blue
- H5: `=I5-1` → 2022, format `0"A"`, border: bottom medium brand blue
- I5: `=Inputs!$F$97` → first forecast year, format `0"F"`, **GREEN font** (`FF006100`, cross-tab ref per REQ-G-110), border: top thin + bottom medium brand blue
- J5-M5: `=prev+1`, format `0"F"`, border: top thin + bottom medium brand blue
- N5: `'Term'` (string), format `0"F"`, border: top thin + bottom medium brand blue
- All year cells: 10pt bold, h=right

### Row 6: Scenario Display
- B6: `=CONCATENATE("Model Running: ",INDEX(Inputs!$B$12:$B$14,Inputs!$D$7)," Drivers")` — 9pt italic, array formula, **GREEN font** (cross-tab ref)

## 11 Schedule Headers in Model

Each schedule header row follows the same pattern as Row 3 (14pt bold `FF3271D2`, `FFE7F2FF` fill).

| # | Row | Header Text (exact) |
|---|-----|-------------------|
| 1 | 3 | Revenue Schedule |
| 2 | 47 | Cost Schedule |
| 3 | 93 | Income Statement |
| 4 | 127 | Working Capital Schedule |
| 5 | 162 | Depreciation Schedule |
| 6 | 208 | Asset Schedule |
| 7 | 237 | Income Tax Schedule (Levered) |
| 8 | 279 | Income Tax Schedule (Unlevered) |
| 9 | 321 | Unlevered Free Cash Flow Schedule |
| 10 | 361 | Discounted Cash Flow Schedule: Perpetuity Method |
| 11 | 408 | Discounted Cash Flow Schedule: Multiple Method |

Each header has a sub-header row (header_row + 1) and a year header row (header_row + 2) following the same pattern as rows 4-5. Each has a units/scenario info row (header_row + 3) like row 6.

**Between schedules:** At least 1 blank row separating the last data row of one schedule from the header row of the next (per REQ-G-014, REQ-G-187).

**Navigation column A:** Place a space character `' '` at each schedule header row in column A for CTRL+Up/Down navigation (per REQ-G-062).

## Row Constants (row_map.py)

```python
# === ROW MAP — All row constants for Model sheet ===
# Verified against extract_complete_excel_SCRIPT.md
# Row numbers are ABSOLUTE (not relative offsets)

# --- Global ---
BANNER_ROW = 1
NAV_ROW = 2

# --- Schedule 1: Revenue (rows 3-44) ---
REV_HEADER = 3
REV_SUBHEADER = 4
REV_YEAR_ROW = 5
REV_INFO_ROW = 6
# ... (detail rows populated in step_05)

# --- Schedule 2: Cost (rows 47-90) ---
COST_HEADER = 47
# ... (detail rows populated in step_06)

# --- Schedule 3: Income Statement (rows 93-124) ---
IS_HEADER = 93
# ... (detail rows populated in step_07)

# --- Schedule 4: Working Capital (rows 127-159) ---
WC_HEADER = 127
# ... (detail rows populated in step_08)

# --- Schedule 5: Depreciation (rows 162-205) ---
DEPR_HEADER = 162
# ... (detail rows populated in step_09)

# --- Schedule 6: Asset Schedule (rows 208-234) ---
ASSET_HEADER = 208
# ... (detail rows populated in step_09b)

# --- Schedule 7: Levered Tax (rows 237-276) ---
LEV_TAX_HEADER = 237
# ... (detail rows populated in step_11)

# --- Schedule 8: Unlevered Tax (rows 279-318) ---
UNLEV_TAX_HEADER = 279
# ... (detail rows populated in step_12)

# --- Schedule 9: UFCF (rows 321-358) ---
UFCF_HEADER = 321
# ... (detail rows populated in step_13)

# --- Schedule 10: DCF Perpetuity (rows 361-405) ---
DCF_PERP_HEADER = 361
# ... (detail rows populated in step_14)

# --- Schedule 11: DCF Multiple (rows 408-452) ---
DCF_MULT_HEADER = 408
# ... (detail rows populated in step_15)
```

**Note:** Detail row constants within each schedule are added by the step that builds that schedule. Step 1 only defines header rows. This prevents dangerous arithmetic offsets (Known Mistake #1, #2 root cause).

## Freeze Panes
- Cover: None
- Outputs: `A2`
- Inputs: `A2`
- Model: `A2`

**NOT `E2`.** Extraction-verified: only row 1 (banner) is frozen, columns are NOT frozen.

## Page Footers (Outputs, Inputs, Model — NOT Cover)
- Left: `'DCF Valuation Modeling'` (font: Open Sans Bold 10, color `002060`)
- Center: `'Page &P of &N'` (same font)
- Right: `'&G'` (graphic/logo)

## Print Areas
- Cover: `$B$2:$N$39`
- Outputs: `$B$3:$Q$44,$B$47:$Q$88`
- Inputs: `$B$3:$M$41,$B$44:$M$85,$B$88:$M$113`
- Model: `$B$3:$O$44,$B$47:$O$90,$B$93:$O$124,$B$127:$O$159,$B$162:$M$205,$B$208:$M$234,$B$237:$M$276,$B$279:$M$318,$B$321:$O$358,$B$361:$O$405,$B$408:$O$452`
  - **Note:** Schedules 5-8 (Depreciation, Asset, Lev Tax, Unlev Tax) extend to $M$ only, NOT $O$

## Other Sheet Properties
- Grid lines: OFF on all sheets
- Iterative calculation: ON (iteration=True, fullCalcOnLoad=True)
- Page orientation: Landscape on all
- Print scale: Cover=77, Outputs/Inputs/Model=88
- Protection: sheet=False, flags set (formatCells=True, formatColumns=True, etc.)
- Default row height: Cover=19.5, Outputs=16.5, Inputs/Model=15.0
- Margins: Cover=0.315" all sides; Outputs/Inputs/Model=0.118" all sides
- Zoom (sheet layout view): Cover=85, Outputs=95, Inputs=100, Model=140
- Cover: Fit to page=True
- Named styles to create: Normal, Comma, Percent, Normal 2 2 2, Hyperlink 2 2, Hyperlink, Comma 2

## Row Grouping (REQ-G-064 — MANDATORY)
All detail rows within each schedule on Outputs, Inputs, and Model must have OutlineLevel=1. This enables collapse/expand navigation per institutional guidelines.

- Model (includes trailing spacer row per group):
  - 4-45 (Rev), 48-91 (Cost), 94-125 (IS), 128-160 (WC), 163-206 (Depr), 209-235 (Asset), 238-277 (Lev Tax), 280-319 (Unlev Tax), 322-359 (UFCF), 362-406 (DCF Perp), 409-453 (DCF Mult)
- Outputs: rows 4-45 and 48-89 — OutlineLevel=1
- Inputs: rows 4-42, 45-86, 89-114 — OutlineLevel=1

## Inputs Column Layout (DIFFERENT from Model — varies per section!)

Each Inputs section has its own column layout:

| Section | Historical Cols | Forecast Cols | Terminal |
|---------|----------------|---------------|---------|
| Drivers (row 3) | **NONE** — G-K forecast only | G-K (format `0"F"`) | L="Term" |
| WACC (row 44) | F-H (format `0"A"`) | I-L (format `0"F"`) | — |
| Other Inputs (row 88) | F-H (format `0"A"`) | I-M (format `0"F"`) | — |

**Builder must NOT use Model's DATA_COLS (F-N) for Inputs sheet. Each section differs.**

## Conditional Formatting (exact formulas from extraction)
- Cover M15:M21: formula `M15=1`, fill bg=`FF947131`, priority 1
- Inputs F98:F99 F110: formula `F98=1`, priority 1
- Model D246: formula `$D$246=1`, priority 3
- Model D288: formula `$D$288=1`, priority 2
- Model I43:N43: formula `I$43=1` (row-locked $), priority 4
- Model I353:N353: formula `I353=1` (NOT locked), priority 1

## Inputs Sheet Structure
3 section headers (**similar but NOT identical formatting to Model** — Inputs D3/E3 are bold unlike Model; Inputs sub-header B4 uses `FF0000FF` pure blue, not `FF3271D2` brand blue):

| Row | Header Text |
|-----|------------|
| 3 | Drivers |
| 44 | Weighted Average Cost of Capital |
| 88 | Other Inputs |

## Outputs Sheet Structure
2 section headers + dual-panel layout:

| Row | Header Text |
|-----|------------|
| 3 | Dashboard: Perpetuity Method |
| 47 | Dashboard: Multiple Method |

Merged ranges: `B68:B72`, `B24:B28`, `J68:J72`, `B81:B85`, `J81:J85`, `J24:J28`, `B37:B41`, `J37:J41`
Manual page break: row=46

## Audit Checklist (openpyxl-verifiable)

1. **Tabs:** `wb.sheetnames == ['Cover', 'Outputs', 'Inputs', 'Model']`
2. **Column widths:** Verify per-sheet widths match extraction (Model B=17.71, NOT v3's 4.0)
3. **Freeze panes:** `ws.freeze_panes == 'A2'` for Outputs/Inputs/Model; None for Cover
4. **Year headers at row 5:** F5 formula `=G5-1`, I5 formula `=Inputs!$F$97`, N5 value `'Term'`
5. **Year formats:** F5-H5 = `0"A"`, I5-N5 = `0"F"`
6. **Year borders:** F5-H5 bottom=medium brand blue; I5-N5 top=thin + bottom=medium brand blue
7. **11 schedule headers:** Verify B{row}.value matches exact text for all 11 rows
8. **Schedule header formatting:** 14pt bold `FF3271D2` font, `FFE7F2FF` fill on each header row
9. **Banner row:** Row 1 fill = `FF000C3F` on Outputs/Inputs/Model (NOT Cover — Cover banner is rows 2-9)
10. **Images:** 4 images total (one per sheet), size 16200x4708
11. **Footers:** Outputs/Inputs/Model have left/center/right footer text
12. **Print areas:** All print area strings match extraction exactly
13. **Grid lines:** OFF on all 4 sheets
14. **Navigation:** Column A has space char at each schedule header row
15. **row_map.py:** File exists, header row constants match extraction rows
16. **No data yet:** All data cells (F-N, below year headers) are empty
17. **Font:** Every cell uses Open Sans (not Calibri/Arial)
18. **Iterative calc:** `wb.calculation.iterate == True`
19. **Row grouping:** Detail rows within each schedule have OutlineLevel=1
20. **Margins:** Cover margins=0.315, others=0.118
21. **Zoom:** Layout view zoom matches per-sheet values (Cover=85, Outputs=95, Inputs=100, Model=140)
22. **Named styles:** Comma 2 and other named styles exist in workbook
23. **Inputs columns:** Drivers=G-L, WACC=F-L, Other Inputs=F-M (NOT Model's F-N)
24. **Green font:** I5 (cross-tab ref `=Inputs!$F$97`) has green font `FF006100`
25. **Cover banner:** Fill on rows 2-9 (NOT row 1)

## Known Traps
- Using `FF0000CC` for blue font instead of `FF3271D2` (v3 error)
- **Not using green font (`FF006100`) for cross-tab links** — Guidelines REQUIRE it (REQ-G-110), template skips it
- Setting freeze to `E2` instead of `A2` (v3 error)
- Using `thin` borders where `hair` is correct (most common grid border)
- Using `double` borders for totals (does NOT exist in reference template)
- 4 historical cols instead of 3 (v3 says F-I historical, actual is F-H)
- Wrong column widths (v3 uses Cover widths for all sheets — each sheet differs)
- Missing row 2 navigation formatting (orange `FFFA621C`)
- Not placing images on all 4 sheets (v3 only mentions Cover)
- 11 schedules not 12 (no standalone "Debt Schedule" in reference template)
- Using lower case or Title Case for section sub-headers instead of UPPER CASE (REQ-G-125)
