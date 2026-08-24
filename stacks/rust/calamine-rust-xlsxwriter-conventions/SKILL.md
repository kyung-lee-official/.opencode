---
name: calamine-rust-xlsxwriter-conventions
description: Use when editing Rust `.xlsx` import/export code — read with `calamine` (prefer display/text form over raw typed cells), write with `rust_xlsxwriter` (freeze top row + autofilter on every data sheet, numeric cells + `num_format` for money/rates), keep high-precision `Decimal` in domain and convert at the export boundary only.
---

# Excel `.xlsx` — calamine + rust_xlsxwriter

Peer of **`exceljs-xlsx-conventions`** for the TypeScript stack.

| Direction                              | Crate                                                                                                      | Notes                                                    |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| **Read** (imports)                     | **calamine**                                                                                               | Lazy, read-only — do not use `rust_xlsxwriter` to read   |
| **Write** (exports / new files)        | **rust_xlsxwriter**                                                                                        | Create/download workbooks — do not use calamine to write |
| **Read → edit → save** existing file   | Prefer **umya-spreadsheet** only when that workflow is required; otherwise keep calamine + rust_xlsxwriter |

---

## Reading imported workbooks (calamine)

When **reading** user-supplied workbook data, prefer the cell's **display /
string form** (trim / normalize as needed) over raw typed variants for
generic labels/codes.

- Formula and rich types are awkward as raw values; prefer what the user sees
  (formatted string), then parse intentionally.
- Use typed cells (`Data::Float`, dates, etc.) only when the task explicitly
  requires it. Add a short comment at the call site.

```rust
// ✅ GOOD — string-ish / trimmed for generic import fields
let s = cell_string.trim(); // from calamine cell → display string helper

// ❌ BAD for generic import fields — blindly taking raw typed value without intent
let raw = match cell { Data::Float(f) => f.to_string(), /* ... */ };
```

Money/domain parsing after read still follows **`rust-convention`** /
`coding-convention` (no `f64` as the long-lived domain money type).

---

## Exporting workbooks (`rust_xlsxwriter`)

On **every data sheet** after headers and body rows are written (before
`workbook.save` / `save_to_buffer`):

1. **Freeze the first row** (header visible while scrolling).
2. **Enable autofilter** covering the header through the data block (at
   least the used column span).

Assumes **row 0** is the header (0-based API). Prefer a shared export helper
if the project has one:

```rust
use rust_xlsxwriter::{Format, Workbook};

// After writing header + body on `worksheet`:
worksheet.set_freeze_panes(1, 0)?; // freeze top row
let last_row = /* last data row index */;
let last_col = /* last column index */;
worksheet.autofilter(0, 0, last_row, last_col)?;
```

Call on **each** tabular sheet in multi-sheet workbooks. Skip freeze/filter
only when layout explicitly requires it (document why). If the header is not
row 0, adjust freeze/autofilter ranges to match.

### Financial and summable amounts

Write **numeric** cells (`write_number` / `write_number_with_format`), not
strings, for money, rates, weights, counts, or any column users should
**`SUM`**. Use **`Format::set_num_format`** for display.

Domain and DB keep high-precision decimals (see **`rust-convention`** —
`rust_decimal::Decimal` or minor units). Convert **at the export boundary**
only: apply an **explicit** round for that column's **display** contract,
then write a number suitable for Excel. Do **not** `write_string` for amount
columns. Excel rounding is presentation — it does not redefine storage
scale.

```rust
let money_fmt = Format::new().set_num_format("#,##0.00");
let rate_fmt = Format::new().set_num_format("0.000000");

// ✅ numeric + format — Excel can SUM
worksheet.write_number_with_format(row, amount_col, amount_display, &money_fmt)?;
worksheet.write_number_with_format(row, rate_col, rate_display, &rate_fmt)?;

// ❌ BAD — SUM() skips / mishandles string cells
worksheet.write_string(row, amount_col, &amount.to_string())?;
```

| Kind                      | Export value                                                        | Typical `num_format`                |
| ------------------------- | ------------------------------------------------------------------- | ----------------------------------- |
| **Money (display)**       | round only if the sheet's display contract requires it, then number | `#,##0.00` or match display dp      |
| **Precise amount / rate** | number at stored scale (often 6 dp)                                 | `0.000000` or enough `0`s for scale |
| **Weight / qty**          | number from domain parse/round at export                            | match business dp                   |

Use blank cells for missing amounts. Keep as **string**: identifier codes
(ids, SKUs), raw audit dumps; summable columns use **number** +
`num_format`.