# CFI Formatting Rules (v4 — Extraction-Verified)

**Source of truth:** Deterministic extraction of CFI DCF template (25 fixes, 6 audit rounds).
**All values verified against:** `cfi_extract_complete_excel_SCRIPT.md`

## Font Colors

| Type | Hex (ARGB) | When to Use |
|------|-----------|-------------|
| Hardcoded Input | `FF3271D2` (CFI brand blue) | Raw data from filings, assumptions, opening balances (REQ-G-108) |
| Formula (same-sheet) | `FF000000` or no explicit color | Formulas referencing cells on the SAME sheet (REQ-G-109) |
| Cross-sheet link | `FF006100` (dark green) | ANY formula referencing a DIFFERENT sheet (REQ-G-110) |
| Labels / Row headers | `FF000000` or `theme=1` | Line item names, section labels |
| Section header text | `FF3271D2` | Schedule title text (e.g., "Income Statement") |
| Orange navigation | `FFFA621C` | Model row 2 banner navigation text |
| Cover title | `FF4472C4` | Main title on Cover sheet |
| Cover section headers | `FF132E57` | "Table of Contents", "Model Checks", "Strictly Confidential" |
| Cover ToC links | `FF002060` | Navigation links |
| White on dark | `theme=0` or `FFFFFFFF` | Text on dark navy banner backgrounds |

**CRITICAL: Template omits green font, but CFI Guidelines REQUIRE it (REQ-G-110). We implement the full standard: `FF006100` for all cross-tab links.**

## Number Formats

### Primary (CFI four-section style)

| Data Type | Format String |
|-----------|--------------|
| Currency (thousands) | `_(#,##0_);\(#,##0\);_("–"_);_(@_)` |
| Currency (1 decimal) | `_(#,##0.0_);\(#,##0.0\);_("–"_);_(@_)` |
| Currency (2 decimal) | `_(#,##0.00_);\(#,##0.00\);_("–"_);_(@_)` |
| Percentage (1 dec) | `_(#,##0.0%_);\(#,##0.0%\);_("–"_)_%;_(@_)_%` |
| Percentage (whole) | `_(#,##0%_);\(#,##0%\);_("–"_)_%;_(@_)_%` |
| Multiple (1 dec) | `_(0.0\x_);\(0.0\x\);_("–"_);_(@_)` |
| Multiple (4 dec) | `_(0.0000\x_);\(0.0000\x\);_("–"_);_(@_)` |

Pattern: `_( pos _) ; \( neg \) ; _("–"_) ; _(@_)` — alignment padding, paren negatives, en-dash zero, text handler.

### Secondary

| Data Type | Format String |
|-----------|--------------|
| Year (historical) | `0"A"` |
| Year (forecast) | `0"F"` |
| Check result | `"Yes";"ERROR";"No";"ERROR"` |
| Date | `yyyy/mm/dd;@` |
| Comma style | `_-* #,##0_-;\(#,##0\)_-;_-* "-"_-;_-@_-` |
| Simple thousands | `#,##0_);(#,##0)` |
| Footnoted text | `@\⁽\¹\⁾` through `@\⁽\⁴\⁾` |

## Borders

| Element | Style | openpyxl |
|---------|-------|----------|
| Data cell grid | **Hair** | `Border(left/right/top/bottom=Side(style='hair', color='FF000000'))` |
| Year header bottom | Medium (brand blue) | `Side(style='medium', color='FF3271D2')` |
| Year header top (forecast) | Thin (brand blue) | `Side(style='thin', color='FF3271D2')` |
| Banner top (Cover F-N) | Thick (navy) | `Side(style='thick', color='FF132E57')` |
| Banner left (Cover B) | Medium (dark navy) | `Side(style='medium', color='FF000C3F')` |
| Sensitivity cells | Thin all sides (black) | `Side(style='thin', color='FF000000')` |

**CRITICAL: No `double` border style exists in the template. v3's "double bottom for totals" is WRONG.**

## Column Layout

Columns differ per sheet. See V4_CONTEXT_BRIEF.md Section 2 for full per-sheet widths.

**Model Sheet key columns:**
- A: 9.14 (spacer), B: 17.71 (labels), C: 11.29 (sub-labels), D: 15.71 (units/flags)
- E: 1.71 (narrow gap), F: 11.71 (first hist year), G-N: 10.29 each (data cols)
- O: 1.71 (trailing spacer)

## Sheet Features

| Feature | Sheet | Setting |
|---------|-------|---------|
| Freeze panes | Model | **A2** (row 1 only) |
| Freeze panes | Inputs | **A2** (row 1 only) |
| Freeze panes | Outputs | **A2** (row 1 only) |
| Freeze panes | Cover | None |
| Grid lines | All | **OFF** |
| Font | All | **Open Sans** |
| Images (CFI logo) | All 4 sheets | Cover: (2,3); Others: (1,0) |
| Page footer | Outputs/Inputs/Model | Left: title, Center: page #, Right: logo |
| Data validation | Inputs!F7 | List: `1,2,3` (scenario selector) |
| Row grouping | Outputs, Inputs, Model | Outline level 1 |
| Iterative calc | Workbook | Enabled |

## Tab Order and Colors

| Position | Tab | Color |
|----------|-----|-------|
| 1 (leftmost) | Cover | — (no tab color set) |
| 2 | Outputs | — |
| 3 | Inputs | — |
| 4 | Model | — |

**Note:** No tab colors are set in the extraction. v3's tab colors were fabricated.

## Text Formatting

| Element | Font Size | Bold | Other |
|---------|-----------|------|-------|
| Schedule headers | 14pt | Yes | Brand blue `FF3271D2`, light blue fill `FFE7F2FF` |
| Data rows | 10pt | No | — |
| Total rows | 10pt | Yes | — |
| Year headers | 10pt | Yes | Right-aligned |
| Cover title | 20pt | Yes | `FF4472C4` |
| Cover checks | 12pt | No | `FF000000` |
| Units/notes | 9pt | Italic | — |
| Banner nav (Model row 2) | 11pt | Bold+Italic | Orange `FFFA621C` |

## Conditional Formatting

| Sheet | Range | Condition | Format |
|-------|-------|-----------|--------|
| Cover | M15:M21 | `M15=1` | Fill bg=`FF947131`, white bold font |
| Inputs | F98:F99, F110 | Check expressions | — |
| Model | D246, D288 | Tax loss checks | — |
| Model | I43:N43, I353:N353 | Capacity/UFCF checks | — |

## Sensitivity Table Formatting

- ODD dimensions (5x5 or 7x7)
- Row/column headers: bold, light gray fill
- Center cell (base case): medium blue fill, bold
- All cells: **thin** borders (black) on all sides — NOT hair
- Number format: per-share `_(#,##0.00_);\(#,##0.00\);_("–"_);_(@_)` or EV thousands
