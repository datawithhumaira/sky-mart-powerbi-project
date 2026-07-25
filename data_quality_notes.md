# Data Quality Notes & Decisions

This document records the data quality issues found during the project and the decisions made to handle them — standard practice for a real BI engagement.

## 1. Mixed date formats (Sales files)
`OrderDate` contained multiple formats in the same column (`2025-04-07`, `10-Apr-2025`, `04/17/2025`). Fixed in Power Query using a custom column with `try...otherwise` fallback parsing via `Date.FromText`, then re-typed as Date. Resolved 19% initial error rate down to 0%.

## 2. 359 rows with blank ProductID (FactSales)
Investigation showed these rows had valid OrderID, CustomerID, Quantity, and UnitPrice — i.e., real transactions, just missing the product link, with no alternate identifying column (ProductName/SKU) to recover it from.

**Decision:** Kept these rows in FactSales (excluding them would understate Total Sales). They are filtered out of Product-level visuals only (via a "ProductName is not blank" visual-level filter), so revenue totals stay accurate while product breakdowns stay clean.

## 3. Category naming inconsistencies (DimProducts)
Some category values existed in two forms, e.g. "Electronics" vs "Elec.", "Furniture" vs "Furn." — splitting what should have been one category into two, understating true category totals.

**Decision:** Standardized via Power Query `Replace Values`.

## 4. Product-level return breakdown too granular
Returns by `ProductName` produced ~500 near-unique bars (SKU-level granularity), each with very low counts — not a bug, just too fine-grained to be useful.

**Decision:** Switched the "top returned items" visual to use `Brand` (8 distinct values) instead of `ProductName`, giving a genuinely useful breakdown.

## 5. "On Time Delivery" definition
Source `FactShipments` has no promised/target delivery date field.

**Decision:** Defined "on time" as delivered within 5 days of ship date — an explicit project assumption, not derived from the business. Documented here and in the DAX reference so it's clear to any reviewer.

## 6. Flat/non-varying "Satisfaction by Region" chart
Initial version of this chart showed an identical average (3.9) across every region — a red flag, since real data almost never comes out perfectly flat.

**Root cause:** the chart used `DimStores[Region]`, which has no relationship path to `FactSurvey` (Survey only relates to `DimCustomers`). Power BI silently rendered the grand total repeated on every bar rather than throwing an error.

**Decision:** Switched to `DimCustomers[Province]`, which does have a live relationship to FactSurvey and produced real, varying results. Region-level satisfaction and category-level satisfaction are not available given the current data model (see DAX_measures.md for details) — stated as an explicit limitation rather than forced.

## 7. Flat "Total Profit" repeating across Budget categories
Same pattern as #6: `FactBudget[Category]`/`[Region]` had no relationship to `FactSales`, so Total Profit showed the same grand-total value on every row of the Budget vs Actual table.

**Decision:** Added inactive many-to-many relationships (`DimProducts[Category] → FactBudget[Category]`, `DimStores[Region] → FactBudget[Region]`) and built dedicated measures using `USERELATIONSHIP()` to activate them only where needed, without disturbing the rest of the model.

## 8. Budget Variance % (~398%) is unusually large
Actual profit consistently exceeded budget targets by roughly 4x across all months, regions, and categories.

**Decision:** Verified this wasn't a scale/row-duplication bug — FactBudget has exactly 144 rows (6 months × 4 regions × 6 categories), matching expectations, and the totals aggregate correctly. Reported as a genuine finding rather than corrected or hidden: either the business significantly outperformed expectations, or budget targets were set conservatively during planning. Flagged directly on the Budget vs Actual page as a callout for stakeholders.

## 9. Profit Margin % definition
Defined in this project as `Total Profit ÷ Total Cost` (markup) rather than the more standard `Total Profit ÷ Total Sales` (margin), by project decision. Documented here and in DAX_measures.md so it isn't mistaken for an error by a reviewer.
