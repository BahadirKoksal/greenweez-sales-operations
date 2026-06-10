-- ============================================================
-- Greenweez Sales Operations Analysis
-- BigQuery SQL Project
-- ============================================================


-- ============================================================
-- STEP 1: Explore raw sales data
-- ============================================================

SELECT *
FROM course17.gwz_sales_17;


-- ============================================================
-- STEP 2: Order-level aggregation
-- Group product lines into orders with total turnover & margin
-- ============================================================

CREATE OR REPLACE TABLE `course17.gwz_orders` AS
SELECT
  date_date,
  orders_id,
  ROUND(SUM(turnover), 2) AS turnover,
  ROUND(SUM(turnover - purchase_cost), 2) AS margin
FROM course17.gwz_sales_17
GROUP BY
  date_date,
  orders_id;


-- ============================================================
-- STEP 3: Add operational costs (shipping + logistics)
-- Join orders with shipping table to add cost breakdown
-- ============================================================

CREATE OR REPLACE TABLE course17.gwz_orders_operational AS
SELECT
  o.date_date,
  o.orders_id,
  o.turnover,
  o.margin,
  sh.shipping_fee,
  sh.log_cost + sh.ship_cost AS operational_cost
FROM course17.gwz_orders AS o
LEFT JOIN course17.gwz_ship_17 AS sh
  ON o.orders_id = sh.orders_id;

-- Preview shipping table
SELECT *
FROM course17.gwz_ship_17;


-- ============================================================
-- STEP 4: Refactor using CTE (cleaner single query)
-- Combines order aggregation + shipping join in one step
-- ============================================================

-- We first grouped sales into gwz_orders, then joined with gwz_ship_17.
-- Here we do both in a single query using a CTE.

CREATE OR REPLACE TABLE course17.gwz_orders_join AS
WITH orders_join AS (
  SELECT
    date_date,
    orders_id,
    ROUND(SUM(turnover), 2) AS turnover,
    ROUND(SUM(turnover - purchase_cost), 2) AS margin
  FROM course17.gwz_sales_17
  GROUP BY
    date_date,
    orders_id
)
SELECT
  o.date_date,
  o.orders_id,
  o.turnover,
  o.margin,
  sh.shipping_fee,
  sh.log_cost + sh.ship_cost AS operational_cost
FROM orders_join AS o
LEFT JOIN course17.gwz_ship_17 AS sh
  ON o.orders_id = sh.orders_id;

-- We joined the financial order table (gwz_orders) with the shipping/operations
-- table (gwz_ship_17) to see not just the sales side, but also the operational costs.

SELECT *
FROM course17.gwz_orders_join
LIMIT 10;


-- ============================================================
-- STEP 5: Daily aggregation + ad spend join
-- Granularity shift: Line-level → Order-level → Date-level
-- ============================================================

-- Ad spend data is tracked at the day level, not the order level.
-- So we aggregate orders by date_date, then join with the campaign table.
-- Step 1: Aggregate orders by date — sum turnover, margin, shipping, operational cost.
-- Step 2: Aggregate ad spend by date from gwz_campaign_17.
-- Step 3: Join both daily tables on date_date.
-- Goal: See for each day — total revenue, margin, operational cost, and ad spend.
-- This is a CFO-level daily P&L view.
-- This granularity control pattern is the backbone of SQL analytics:
-- Line-level → Order-level → Date-level

SELECT *
FROM course17.gwz_campaign_join
LIMIT 10;


-- ============================================================
-- STEP 6: Margin level classification
-- Chained CTEs + CASE WHEN to segment products by margin
-- ============================================================

-- CTE 1: Calculate raw margin per product line
-- CTE 2: Calculate margin as a percentage of turnover (SAFE_DIVIDE to avoid division by zero)
-- Final SELECT: Classify each line as low / medium / high margin

WITH margin_table AS (
  SELECT
    orders_id,
    products_id,
    turnover,
    turnover - purchase_cost AS margin
  FROM course17.gwz_sales_17
),

margin_percent_table AS (
  SELECT
    orders_id,
    products_id,
    turnover,
    margin,
    ROUND(SAFE_DIVIDE(margin, turnover), 2) AS margin_percent
  FROM margin_table
)

SELECT
  orders_id,
  products_id,
  turnover,
  margin,
  margin_percent,
  CASE
    WHEN margin_percent < 0.05  THEN "low"
    WHEN margin_percent > 0.40  THEN "high"
    WHEN margin_percent BETWEEN 0.05 AND 0.40 THEN "medium"
    ELSE NULL
  END AS margin_level
FROM margin_percent_table;


-- ============================================================
-- STEP 7: Promotion type classification
-- Classify promotions based on discount multiplier
-- ============================================================

-- Uses CASE WHEN to assign promo_type:
-- Short-lived, Low promotion, or High promotion
-- Based on the promo multiplier and promo_percent values
