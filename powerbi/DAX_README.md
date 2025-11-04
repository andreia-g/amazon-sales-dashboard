# Amazon Sales Dashboard – Power BI DAX Measures

This README lists the exact DAX measures used in the Amazon Sales Dashboard Power BI model, matching the JSON export. All measures are stored in the display folder **DAX Formulas**.

- Table: `Amazon_Sales_cleaned`
- Date column: `Amazon_Sales_cleaned[date]`
- Amount column: `Amazon_Sales_cleaned[amount]`
- Quantity column: `Amazon_Sales_cleaned[qty]`
- Order ID column: `Amazon_Sales_cleaned[order_id]`
- Status column: `Amazon_Sales_cleaned[status]`
- Fulfillment column: `Amazon_Sales_cleaned[fulfilment]` (text: "Amazon" / "Merchant")
- B2B column: `Amazon_Sales_cleaned[b2b]` (boolean)
- Promotion flag column: `Amazon_Sales_cleaned[has_promotion]` (0/1)
- Engineered columns: `order_status_category`, `size_category`, `unit_price`, `is_weekend`

---

## Base Measures

```
Total Revenue = SUM(Amazon_Sales_cleaned[amount])
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

Total Quantity = SUM(Amazon_Sales_cleaned[qty])
Format: #,0

Total Orders = DISTINCTCOUNT(Amazon_Sales_cleaned[order_id])
Format: #,0

Average Order Value (AOV) = DIVIDE([Total Revenue], [Total Orders])
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00
```

---

## Status Measures (using order_status_category)

```
Cancelled Orders = 
COUNTROWS(FILTER(Amazon_Sales_cleaned, Amazon_Sales_cleaned[order_status_category] = "Cancelled"))
Format: #,0

Completed Orders = 
COUNTROWS(FILTER(Amazon_Sales_cleaned, Amazon_Sales_cleaned[order_status_category] = "Completed"))
Format: #,0

Cancellation Rate % = 
DIVIDE(
    CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[order_status_category] = "Cancelled"),
    COUNTROWS(Amazon_Sales_cleaned)
)
Format: 0.00%;-0.00%;0.00%

Completion Rate % = 
DIVIDE(
    CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[order_status_category] = "Completed"),
    COUNTROWS(Amazon_Sales_cleaned)
)
Format: 0.00%;-0.00%;0.00%
```

---

## Fulfillment Measures

```
Amazon Orders = 
CALCULATE(COUNTROWS(Amazon_Sales_cleaned), LOWER(Amazon_Sales_cleaned[fulfilment]) = "amazon")
Format: #,0

Merchant Orders = 
CALCULATE(COUNTROWS(Amazon_Sales_cleaned), LOWER(Amazon_Sales_cleaned[fulfilment]) = "merchant")
Format: #,0

Amazon Fulfillment % = 
DIVIDE([Amazon Orders], [Amazon Orders] + [Merchant Orders])
Format: 0.00%;-0.00%;0.00%

Amazon Revenue = 
CALCULATE([Total Revenue], LOWER(Amazon_Sales_cleaned[fulfilment]) = "amazon")
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

Merchant Revenue = 
CALCULATE([Total Revenue], LOWER(Amazon_Sales_cleaned[fulfilment]) = "merchant")
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00
```

---

## B2B / B2C Measures

```
B2B Orders = 
CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[b2b] = TRUE())
Format: #,0

B2C Orders = 
CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[b2b] = FALSE())
Format: #,0

B2B Revenue = 
CALCULATE([Total Revenue], Amazon_Sales_cleaned[b2b] = TRUE())
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

B2C Revenue = [Total Revenue] - [B2B Revenue]
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

B2B Revenue % = 
DIVIDE([B2B Revenue], [Total Revenue])
Format: 0.00%;-0.00%;0.00%

B2B AOV = 
DIVIDE(
    CALCULATE([Total Revenue], Amazon_Sales_cleaned[b2b] = TRUE()),
    CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[b2b] = TRUE())
)
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

B2C AOV = 
DIVIDE(
    CALCULATE([Total Revenue], Amazon_Sales_cleaned[b2b] = FALSE()),
    CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[b2b] = FALSE())
)
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00
```

---

## Promotion Measures

```
Promo Orders = 
CALCULATE(COUNTROWS(Amazon_Sales_cleaned), Amazon_Sales_cleaned[has_promotion] = 1)
Format: 0

Non-Promo Orders = [Total Orders] - [Promo Orders]
Format: #,0

Promo Revenue = 
CALCULATE([Total Revenue], Amazon_Sales_cleaned[has_promotion] = 1)
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

Non-Promo Revenue = [Total Revenue] - [Promo Revenue]
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

Promo Share of Orders % = 
DIVIDE([Promo Orders], [Total Orders])
Format: 0.00%;-0.00%;0.00%

Promo Revenue % = 
DIVIDE([Promo Revenue], [Total Revenue])
Format: 0.00%;-0.00%;0.00%
```

---

## Weekend / Weekday Measures

```
Weekend Revenue = 
CALCULATE([Total Revenue], Amazon_Sales_cleaned[is_weekend] = 1)
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

Weekday Revenue = [Total Revenue] - [Weekend Revenue]
Format: "₹"#,0.00;#,0.00\ -"₹";"₹"#,0.00

Weekend Share % = 
DIVIDE([Weekday Revenue], [Total Revenue])
Format: 0.00%;-0.00%;0.00%
```

---

## Time Intelligence Measures (No DimDate)

Using the fact table date directly. For advanced time intelligence, consider adding a dedicated Date table.

```
Revenue This Month = 
CALCULATE([Total Revenue], DATESMTD(Amazon_Sales_cleaned[date]))

Revenue Last Month = 
VAR startPrev = DATE(YEAR(TODAY()), MONTH(TODAY())-1, 1)
VAR endPrev   = EOMONTH(TODAY(), -1)
RETURN CALCULATE([Total Revenue], Amazon_Sales_cleaned[date] >= startPrev && Amazon_Sales_cleaned[date] <= endPrev)

MoM Revenue % = 
DIVIDE([Revenue This Month] - [Revenue Last Month], [Revenue Last Month])
```

---

## Category / Size / SKU Measures

These work with the engineered columns from the Python notebook.

```
Revenue by Category = [Total Revenue]
-- Use with Category axis

Orders by Size Category = DISTINCTCOUNT(Amazon_Sales_cleaned[order_id])
-- Use with size_category axis

Top SKU Revenue = [Total Revenue]
-- Use with SKU axis; apply Top N filter

Unit Price (Avg) = AVERAGE(Amazon_Sales_cleaned[unit_price])
-- Average unit price across all orders
```

---

## Geography Measures

```
Revenue by State = [Total Revenue]
-- Use with ship_state axis

Orders by City = DISTINCTCOUNT(Amazon_Sales_cleaned[order_id])
-- Use with ship_city axis
```

---

## Operational Measures

```
Cancellation Rate % by Month = [Cancellation Rate %]
-- Use with order_month_name axis

Status Count = COUNTROWS(Amazon_Sales_cleaned)
-- Use with order_status_category legend for stacked charts
```

---

## KPI Card Measures (Summary)

- Total Revenue → `[Total Revenue]`
- Total Orders → `[Total Orders]`
- Average Order Value → `[Average Order Value (AOV)]`
- Cancellation Rate % → `[Cancellation Rate %]`
- Amazon Fulfillment % → `[Amazon Fulfillment %]`
- B2B Revenue % → `[B2B Revenue %]`

---

## Notes & Tips

- No DimDate table is required; measures use `Amazon_Sales_cleaned[date]` directly.
- All currency measures use INR formatting (`"₹"#,0.00`) with Hindi culture hint.
- Percent measures use `0.00%` format for consistency.
- Text-based filters use `LOWER()` for case-insensitive matching.
- For month axis sorting, ensure `order_month_name` is sorted by a numeric `YearMonth` column.
- All measures are stored in the **DAX Formulas** display folder for organization.
- The model includes engineered columns from Python (e.g., `order_status_category`, `is_weekend`, `has_promotion`) to simplify DAX logic.

---

## Total Measures Count

- **Base:** 4 measures
- **Status:** 4 measures  
- **Fulfillment:** 5 measures
- **B2B:** 8 measures
- **Promotions:** 6 measures
- **Weekend:** 3 measures
- **Time Intelligence:** 3 measures
- **Product/Category:** 4 measures
- **Geography:** 2 measures
- **Operations:** 2 measures

**Total: 41 measures** (including KPI references)
