# UK E-Commerce SQL Analysis

> **Tools:** SQL (Google BigQuery) · Business Analytics

Extended the UK E-Commerce dashboard project with SQL to answer deeper
business questions directly from raw transaction data.

## Key Insights
- UK generated **£16.38M** in revenue (49,108 orders) — 26x the next
  market (EIRE) — but the business reaches 37+ countries
- **75.41% repeat customer rate** (4,481 of 5,942 customers) — strong
  customer loyalty signal
- **Saturday orders are near-zero** (32 vs. 11,335 on Thursday) — likely
  no Saturday operations
- Revenue peaks in **November** (~£1.42M), consistent with holiday demand
- **Shipping/postage** line items rank in the top 10 revenue generators —
  a meaningful revenue stream, not just a cost

## Business Questions & Queries

### 1. Top 10 best-selling products by revenue
```sql
SELECT Description, SUM(Quantity * Price) AS total_revenue, SUM(Quantity) AS total_units
FROM `uk-e-502509.uk_ecommerce.sales`
GROUP BY Description
ORDER BY total_revenue DESC
LIMIT 10;
```
**Result:** Regency Cakestand 3 Tier leads (£327,813), followed by Dotcom Postage (£322,647).

![Top Products](screenshots/01_top_products.png)

---

### 2. Monthly revenue trend
```sql
SELECT FORMAT_DATE('%Y-%m', DATE(InvoiceDate)) AS month, SUM(Quantity * Price) AS total_revenue
FROM `uk-e-502509.uk_ecommerce.sales`
GROUP BY month
ORDER BY month;
```
**Result:** Revenue peaks in Nov 2010 (£1.42M), consistent with holiday shopping season.

![Monthly Trend](screenshots/02_monthly_trend.png)

---

### 3. Top 10 countries by revenue
```sql
SELECT Country, SUM(Quantity * Price) AS total_revenue, COUNT(DISTINCT Invoice) AS total_orders
FROM `uk-e-502509.uk_ecommerce.sales`
GROUP BY Country
ORDER BY total_revenue DESC
LIMIT 10;
```
**Result:** UK dominates (£16.38M); EIRE, Netherlands, and Germany follow at far lower volumes.

![Top Countries](screenshots/03_top_countries.png)

---

### 4. Repeat customer rate
```sql
SELECT COUNT(DISTINCT CustomerID) AS total_customers,
       COUNT(DISTINCT CASE WHEN order_count > 1 THEN CustomerID END) AS repeat_customers,
       ROUND(COUNT(DISTINCT CASE WHEN order_count > 1 THEN CustomerID END) / COUNT(DISTINCT CustomerID) * 100, 2) AS repeat_rate_pct
FROM (
  SELECT CustomerID, COUNT(DISTINCT Invoice) AS order_count
  FROM `uk-e-502509.uk_ecommerce.sales`
  WHERE CustomerID IS NOT NULL
  GROUP BY CustomerID
);
```
**Result:** 75.41% repeat purchase rate (4,481 of 5,942 customers).

![Repeat Rate](screenshots/04_repeat_rate.png)

---

### 5. Average order value by month
```sql
SELECT FORMAT_DATE('%Y-%m', DATE(InvoiceDate)) AS month, ROUND(SUM(Quantity * Price) / COUNT(DISTINCT Invoice), 2) AS avg_order_value
FROM `uk-e-502509.uk_ecommerce.sales`
GROUP BY month
ORDER BY month;
```
**Result:** AOV fluctuates between ~£254–£388, with no extreme seasonal spike in order size — revenue growth is driven more by order volume than order size.

![AOV](screenshots/05_aov.png)

---

### 6. Top 10 customers by total spend
```sql
SELECT CustomerID, SUM(Quantity * Price) AS total_spend, COUNT(DISTINCT Invoice) AS total_orders
FROM `uk-e-502509.uk_ecommerce.sales`
WHERE CustomerID IS NOT NULL
GROUP BY CustomerID
ORDER BY total_spend DESC
LIMIT 10;
```
**Result:** Top customer (ID 18102) spent ~£598K across 153 orders — notable revenue concentration among a small group of customers.

![Top Customers](screenshots/06_top_customers.png)

---

### 7. Lowest-revenue products (still sold)
```sql
SELECT Description, SUM(Quantity * Price) AS total_revenue
FROM `uk-e-502509.uk_ecommerce.sales`
GROUP BY Description
HAVING total_revenue > 0
ORDER BY total_revenue ASC
LIMIT 10;
```
**Result:** Mostly low-cost novelty items (mugs, decorations) under £1 in total revenue — candidates for catalog pruning.

![Worst Products](screenshots/07_worst_products.png)

---

### 8. Order volume by day of week
```sql
SELECT FORMAT_DATE('%A', DATE(InvoiceDate)) AS day_of_week, COUNT(DISTINCT Invoice) AS total_orders
FROM `uk-e-502509.uk_ecommerce.sales`
GROUP BY day_of_week
ORDER BY total_orders DESC;
```
**Result:** Thursday is the peak order day (11,335); Saturday is nearly zero (32) — likely no Saturday operations.

![Orders by Day](screenshots/08_orders_by_day.png)

---

## Author
Mubeen Nadeem — [LinkedIn](https://www.linkedin.com/in/mubeen-nadeem-9845562a2/) | [GitHub](https://github.com/mubeenadeem) | [Portfolio](https://mubeenadeem.github.io)
