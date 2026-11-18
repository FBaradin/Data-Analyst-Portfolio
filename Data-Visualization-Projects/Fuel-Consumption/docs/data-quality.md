# Data Quality Notes

Source: `data/fuel-log.xlsx`, sheet **Combustible**, Excel table **Tabla1**.
149 fill-ups recorded manually between **2022-02-17** and **2026-10-27**.

## Validation performed

All checks run in the `QA_Checks` Power Query query, which is not loaded into the model.

| Check | Result |
|---|---|
| Column quality: errors and empty values | None in any column |
| `Km Driven` (recalculated from the odometer) vs. Excel's `Km recorridos` | All 148 comparable rows match |
| `Days Since Last` (recalculated from dates) vs. Excel's `Cantidad de días` | All 148 comparable rows match |
| `Amount` vs. `Liters × Price per Liter` | 3 rows with significant differences (see below) |

Because both recalculated columns match, the Excel-calculated columns (`Km recorridos`, `Cantidad de días`, `km/L`, `₡/km`) are dropped. The ratios are rebuilt as DAX measures instead.

## Findings

### 1. Amount does not match Liters × Price on 3 rows

Most rows differ by a few colones, at most ₡15. That comes from rounding: the pump displays liters with limited decimals, and amounts are rounded to whole colones. These rows are treated as correct.

Three rows go well beyond rounding:

| Date | Liters | Price (₡/L) | Amount (₡) | Difference (₡) | Liters implied by Amount ÷ Price |
|---|---|---|---|---|---|
| 2023-03-26 | 37.29 | 761 | 28,831 | 453.31 | 37.89 |
| 2024-04-26 | 31.835 | 728 | 25,145 | 1,969.12 | 34.54 |
| 2025-10-15 | 32.5 | 663 | 22,000 | 452.50 | 33.18 |

The price on each of these rows matches the fill-ups just before and after it, so the **liters value is the most likely typo**. On 2023-03-26, for example, 37.89 is a single-digit slip from 37.29. The differences don't repeat in a pattern, which points to data-entry mistakes rather than a systematic cause.

**Decision:** `Amount` is treated as the source of truth, because it is what was actually paid (taken from the receipt or card statement). The 3 rows are kept as recorded and listed here as a known limitation. They affect liter-based measures (km/L, average price per liter) for those three fill-ups only, which is 2% of rows.

### 2. Fuel type has a trailing-space inconsistency

`Tipo combustible` shows two distinct values, `"Súper"` (106 rows) and `"Súper "` (43 rows, with a trailing space). They are the same fuel.

**Resolved:** trimmed in `Refuels_Staging` (Transform → Format → Trim), so the column now has a single value, `Súper`. With only one real fuel type, a separate Fuel Type dimension table adds nothing to the model for now.

## Out of scope (for now)

The workbook also has a **Mantenimiento** sheet (table `Tabla2`, 25 rows of maintenance purchases). It isn't used yet. It could later extend the analysis from fuel cost to total cost of ownership.
