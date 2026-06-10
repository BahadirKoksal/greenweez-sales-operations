# 🛒 Greenweez Sales Operations Analysis

BigQuery SQL project analyzing sales, margin, and operational performance for **Greenweez**, a French organic e-commerce company.

---

## 🎯 Objective

The data team at Greenweez wanted to understand the full financial picture of their sales operations — from raw product-level transactions to a daily P&L view.

This project answers:

- What is the total revenue and margin per order?
- What are the operational (shipping + logistics) costs per order?
- How do daily revenue, margin, and ad spend look together?
- Which products have low, medium, or high margin?
- Which promotions drive the most volume?

---

## 🗂️ Dataset

Three source tables from BigQuery (`course17` dataset):

| Table | Description |
|---|---|
| `gwz_sales_17` | One row per product line — order, product, category, turnover, purchase cost |
| `gwz_ship_17` | One row per order — transporter, shipping fee, log cost, ship cost |
| `gwz_campaign_17` | One row per day — daily ad spend |

**Key columns in `gwz_sales_17`:**

| Column | Type | Description |
|---|---|---|
| `date_date` | DATE | Transaction date |
| `orders_id` | INT | Order identifier |
| `products_id` | INT | Product identifier |
| `customers_id` | INT | Customer identifier |
| `category_1/2/3` | STRING | Product category hierarchy |
| `turnover` | FLOAT | Revenue from this line |
| `purchase_cost` | FLOAT | Cost of goods sold |
| `promo_name` | STRING | Promotion name (if any) |

---

## 🪜 Project Steps

### 1. Data Exploration
Previewed source tables with `SELECT *` to understand structure, volume, and content.
→ `gwz_sales_17` has **1,168,081 rows** (product-level transactions)

### 2. Order-Level Aggregation
Grouped sales by `orders_id` to calculate total turnover and margin per order.
Created table: `gwz_orders` → **142,409 rows**

### 3. Operational Cost Join
Joined `gwz_orders` with `gwz_ship_17` to add shipping fee and logistics cost per order.
Created table: `gwz_orders_operational`

### 4. CTE Refactor
Combined steps 2 and 3 into a single query using a CTE (`WITH` clause).
Created table: `gwz_orders_join` → same result, cleaner code

### 5. Daily Aggregation + Ad Cost Join
Grouped orders by `date_date` and joined with `gwz_campaign_17` to add daily ad spend.
Created table: `gwz_campaign_join` → **153 rows** (one per day)

### 6. Margin Level Classification
Calculated `margin_percent` per product line using `SAFE_DIVIDE`, then classified into levels using `CASE WHEN`.

| Level | Condition |
|---|---|
| `low` | margin_percent < 0.05 |
| `medium` | margin_percent BETWEEN 0.05 AND 0.40 |
| `high` | margin_percent > 0.40 |

### 7. Promotion Classification
Classified promotions into types (`Short-lived`, `Low promotion`, `High promotion`) based on promo multiplier.

---

## 📊 Key Findings

- Some orders have **negative margin** — product cost exceeds revenue on those orders.
- Granularity reduced dramatically across steps: 1,168,081 → 142,409 → 153 rows.
- Daily view enables **ROAS (Return on Ad Spend)** calculation by combining turnover and ad cost.
- Promotion type analysis enables comparison of which campaign types drive volume vs. margin.

---

## 🛠️ SQL Techniques Used

| Technique | Purpose |
|---|---|
| `SELECT / FROM / GROUP BY` | Aggregation and data exploration |
| `SUM / ROUND` | Calculating and rounding totals |
| `LEFT JOIN` | Adding shipping and campaign data to orders |
| `CREATE OR REPLACE TABLE` | Saving intermediate and final tables in BigQuery |
| `WITH` (CTE) | Cleaner multi-step queries without temp tables |
| `SAFE_DIVIDE` | Division without zero-division errors |
| `CASE WHEN` | Classifying margin levels and promo types |
| `LIMIT` | Sampling results during exploration |

---

## 💾 Output Tables

```
course17/
├── gwz_orders              ← Order-level revenue and margin
├── gwz_orders_operational  ← Orders with shipping + logistics cost
├── gwz_orders_join         ← Same as above, built with CTE
└── gwz_campaign_join       ← Daily P&L with ad spend
```

---

## 🔧 Tools

- **Google BigQuery** — SQL engine and data warehouse
- **SQL** — All analysis done in standard SQL with BigQuery-specific functions
