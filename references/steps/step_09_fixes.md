# Step 9 Fix Record: Depreciation Schedule

Template — filled during execution by the fix agent.

## Fix Protocol

For EACH finding from the audit:

### 1. VERIFY the problem independently
Do NOT trust the audit report blindly. Re-read the cell yourself:
```python
wb = openpyxl.load_workbook(xlsx_path, data_only=False)
cell = wb['Model']['{reported_cell}']
print(f"Actual value: {cell.value}")
# Confirm the audit finding is real
```

### 2. PLAN the implementation
Before changing any code:
- Identify the EXACT line in `build_autodcf.py` that produces this cell
- Understand WHY the bug exists (wrong row constant? wrong formula pattern? typo?)
- Check if fixing this line will break any OTHER cells (ripple effects)
- Write the corrected line BEFORE applying it

### 3. APPLY the fix safely
- Edit ONLY the specific line(s) needed
- Re-run the build script
- Verify the xlsx was regenerated

### 4. RE-VERIFY
- Re-run the specific audit check that failed
- Also spot-check 2 adjacent cells (to ensure no ripple damage)

---

## Findings Log

### Finding 1
- **Audit Check**: #{check_number}
- **Cell**: {sheet}!{cell_address}
- **Expected**: {what_should_be_there}
- **Actual**: {what_was_found}
- **Severity**: {CATASTROPHIC|STRUCTURAL|DATA|FORMAT}
- **Verified independently**: {YES/NO}
- **Root cause in build script**: Line {N}: `{code_snippet}`
- **Fix applied**: `{corrected_code}`
- **Re-verified**: {YES/NO}
- **Ripple check**: {cells_checked, all_OK}

### Finding 2
[same template]

---

## Summary
- Findings: {count}
- Fixed: {count}
- Re-verified: {count}
- Status: {ALL_FIXED | REMAINING_ISSUES}
