---
name: exceljs-xlsx-conventions
description: Use when editing ExcelJS `.xlsx` import/export in TypeScript — read with `cell.text` (not `cell.value`), write with freeze pane + autoFilter on every data sheet, numeric cells + `numFmt` for money/rates, and `Decimal` in domain (convert at export boundary only).
---

# ExcelJS — `.xlsx` import and export conventions

## Reading imported workbooks

When **reading** user-supplied workbook data, prefer **`cell.text`** (trim /
normalize as needed) over **`cell.value`**.

- **`cell.value`** for **formulas** is often an object (e.g.
  `{ formula, result }`); `String(value)` breaks parsing.
- **Hyperlinks** and rich types surface awkwardly on **`value`**; **`text`**
  matches what the user sees.

```typescript
// ✅ GOOD
const s = row.getCell(col).text?.trim() ?? "";

// ❌ BAD for generic import fields
const raw = row.getCell(col).value;
```

Use **`cell.value`**, **`cell.hyperlink`**, **`cell.formula`**, or typed
handling only when the task explicitly requires it. Add a short comment at
the call site.

---

## Exporting workbooks for download

On **every data sheet** after headers and body rows are written (before
`workbook.xlsx.writeBuffer()`):

1. **Freeze the first row** (header visible while scrolling).
2. **Enable auto-filter** on the header row.

Assumes **row 1** is the header. Prefer a shared export helper if the project
has one; otherwise set view and filter inline:

```typescript
const columnCount = worksheet.columnCount;
worksheet.views = [{ state: "frozen", ySplit: 1, activeCell: "A2" }];
worksheet.autoFilter = {
  from: { row: 1, column: 1 },
  to: { row: 1, column: columnCount },
};
```

Call on **each** tabular sheet in multi-sheet workbooks. Skip freeze/filter
only when layout explicitly requires it (document why). If header is not
row 1, adjust `ySplit`, `autoFilter`, and `activeCell` to match.

### Financial and summable amounts

Write **numeric** cell values (`number`), not **strings**, for money, rates,
weights, counts, or any column users should **`SUM`**. Use **`numFmt`** for
display.

Domain and DB keep high-precision **`Decimal`** (typically
`@db.Decimal(17, 6)` — see `typescript-convention`). Convert **at the export
boundary** only: apply an **explicit** round for that column's **display**
contract, then **`.toNumber()`**. Do not pass `String(decimal)` or
`decimal.toString()` into **`addRow`** for amount columns. Excel rounding is
presentation — it does not redefine storage scale.

```typescript
// ✅ GOOD — numeric value; Excel can SUM
worksheet.columns = [
  { header: "Amount", key: "amount", width: 15, style: { numFmt: "#,##0.00" } },
  { header: "Rate", key: "rate", width: 15, style: { numFmt: "0.000000" } },
];
worksheet.addRow({
  // Display contract for this sheet: cents; DB may still be Decimal(17, 6)
  amount: item.amount.toDecimalPlaces(2).toNumber(),
  rate: item.rate.toNumber(), // keep stored scale when the sheet shows full precision
});

// ❌ BAD — SUM() skips string cells
worksheet.addRow({ amount: item.amount.toString() });
```

| Kind                      | Export value                                                          | Typical `numFmt`                       |
| ------------------------- | --------------------------------------------------------------------- | -------------------------------------- |
| **Money (display)**       | round only if the sheet's display contract requires it, then `number` | `#,##0.00` or match display dp         |
| **Precise amount / rate** | `number` at stored scale (often 6 dp)                                 | `0.000000` or enough `#`/`0` for scale |
| **Weight / qty**          | `number` from domain parse/round at export                            | match business dp                      |

Use **`null`** / empty for missing amounts. Keep as **string**: identifier
codes (ids, SKUs), raw audit dumps; summable columns use **number** +
`numFmt`.