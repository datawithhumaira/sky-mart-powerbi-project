# DAX Measures Reference

All measures live in the `_Measures` table.

## Question 1 — Sales & Profit Trend
```DAX
Total Sales = SUMX(FactSales, FactSales[Quantity] * FactSales[UnitPrice] * (1 - FactSales[DiscountPct]))

Total Cost = SUMX(FactSales, FactSales[Quantity] * RELATED(DimProducts[UnitCost]))

Total Profit = [Total Sales] - [Total Cost]

Profit Margin % = DIVIDE([Total Profit], [Total Cost])
-- Note: intentionally defined as Profit ÷ Cost (markup), not the standard Profit ÷ Sales margin.
-- Kept this way by project decision; documented here to avoid confusion.

Sales MoM % =
VAR CurrM = [Total Sales]
VAR PrevM = CALCULATE([Total Sales], DATEADD(DimDate[Date], -1, MONTH))
RETURN DIVIDE(CurrM - PrevM, PrevM)

Profit MoM % =
VAR CurrM = [Total Profit]
VAR PrevM = CALCULATE([Total Profit], DATEADD(DimDate[Date], -1, MONTH))
RETURN DIVIDE(CurrM - PrevM, PrevM)
```

## Question 2 — Revenue Drivers
```DAX
Sales Rank (Product) = RANKX(ALL(DimProducts[ProductName]), [Total Sales])

% of Total Sales = DIVIDE([Total Sales], CALCULATE([Total Sales], ALL(DimProducts), ALL(DimStores)))
```

## Question 3 — Returns
```DAX
Total Returns = COUNTROWS(FactReturns)

Total Orders = DISTINCTCOUNT(FactSales[OrderID])

Return Rate % = DIVIDE([Total Returns], [Total Orders])
```

## Question 4 — Inventory
```DAX
Current Stock = SUM(FactInventory[OnHandQty])

Reorder Level = SUM(FactInventory[ReorderLevel])

Stock Status = IF([Current Stock] < [Reorder Level], "Understocked",
               IF([Current Stock] > [Reorder Level] * 2, "Overstocked", "OK"))

-- Calculated column (for use as chart Axis/Legend) in FactInventory:
Stock Status Column =
IF(FactInventory[OnHandQty] < FactInventory[ReorderLevel], "Understocked",
   IF(FactInventory[OnHandQty] > FactInventory[ReorderLevel] * 2, "Overstocked", "OK"))

Understocked Count = CALCULATE(COUNTROWS(FactInventory), FactInventory[Stock Status Column] = "Understocked")
Overstocked Count = CALCULATE(COUNTROWS(FactInventory), FactInventory[Stock Status Column] = "Overstocked")
```

## Question 5 — Shipments
```DAX
Total Shipments = COUNTROWS(FactShipments)

On Time Shipments = CALCULATE(COUNTROWS(FactShipments), FactShipments[DeliveryDate] <= FactShipments[ShipDate] + 5)
-- Assumption: "on time" = delivered within 5 days of ship date.
-- Source data has no PromisedDate/target-delivery field, so this threshold was defined as a project assumption.

On Time Delivery % = DIVIDE([On Time Shipments], [Total Shipments])

Avg Delivery Days = AVERAGEX(FactShipments, DATEDIFF(FactShipments[ShipDate], FactShipments[DeliveryDate], DAY))

-- Calculated column in FactShipments (for use as chart Axis/Legend):
Delivery Status Column =
IF(FactShipments[DeliveryDate] <= FactShipments[ShipDate] + 5, "On Time", "Late")
```

## Question 6 — Customer Satisfaction
```DAX
Avg Satisfaction Score = AVERAGE(FactSurvey[Rating])
```
Breakdown by Region isn't available directly (DimCustomers only has Province — Region lives on DimStores, which has no path to FactSurvey). Province is used as the geographic breakdown instead. Category-level satisfaction is not available in the source data — survey responses are tied to customers, not specific purchases.

## Question 7 — Budget vs Actual
```DAX
Budget Amount = SUM(FactBudget[ProfitTarget])

Budget Variance = [Total Profit] - [Budget Amount]

Budget Variance % = DIVIDE([Budget Variance], [Budget Amount])

Sales Target = SUM(FactBudget[SalesTarget])

Sales Variance vs Target = [Total Sales] - [Sales Target]

Sales Variance vs Target % = DIVIDE([Sales Variance vs Target], [Sales Target])
```

Cross-fact filtering (FactBudget has no natural key relationship to FactSales/FactProducts/FactStores), so two dimension-to-fact bridge relationships were added as **inactive** many-to-many relationships, invoked explicitly via `USERELATIONSHIP()`:

```DAX
Total Profit (by Budget Category) =
CALCULATE([Total Profit], USERELATIONSHIP(DimProducts[Category], FactBudget[Category]))

Total Profit (by Budget Region) =
CALCULATE([Total Profit], USERELATIONSHIP(DimStores[Region], FactBudget[Region]))
```
