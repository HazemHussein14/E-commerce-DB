# SQL Queries Documentation

## Table of Contents

1. [Query 1: Total Revenue for a Specific Date](#query-1-total-revenue-for-a-specific-date)
2. [Query 2: Monthly Top-Selling Products](#query-2-monthly-top-selling-products)
3. [Query 3: High-Value Customers for the Previous Month](#query-3-high-value-customers-for-the-previous-month)
4. [Query 4: Search for Products by Keyword](#query-4-search-for-products-by-keyword)
5. [Query 5: Recommend Products from the Same Category](#query-5-recommend-products-from-the-same-category-excluding-purchased-products)
6. [Query 6: Products per Category](#query-6-products-per-category)
7. [Query 7: Top Customers by Spending](#query-7-top-customers-by-spending)
8. [Query 8: Recent Orders with Customer Information](#query-8-recent-orders-with-customer-information)
9. [Query 9: Low Stock Products](#query-9-low-stock-products)
10. [Query 10: Category Revenue](#query-10-category-revenue)
11. [Performance Optimizations](#performance-optimizations)
    - [Products per Category Optimization](#products-per-category-optimization)
    - [Top Customers Optimization](#top-customers-optimization)
    - [Recent Orders Optimization](#recent-orders-optimization)
    - [Low Stock Optimization](#low-stock-optimization)
    - [Category Revenue Optimization](#category-revenue-optimization)

## Query 1: Total Revenue for a Specific Date

```sql
SELECT
    SUM(total_amount) AS total_revenue
FROM
    "order"
WHERE
    DATE(order_date) = '2024-12-01' -- Replace with the specific date
```

### Description:

This query calculates the total revenue generated on a specific date.

### Usage:

Replace `'2024-12-01'` with the desired date to get the total revenue for that day.

---

## Query 2: Monthly Top-Selling Products

```sql
SELECT
    p.name AS product_name,
    SUM(od.quantity) AS total_quantity_sold,
FROM
    order_details od
JOIN
    product p ON od.product_id = p.product_id
JOIN
    "order" o ON od.order_id = o.order_id
WHERE
    DATE_TRUNC('month', o.order_date) = DATE_TRUNC('month', '2024-12-01'::DATE) -- Replace with the specific date
GROUP BY
    p.product_id, p.name
ORDER BY
    total_quantity_sold DESC
LIMIT 10

```

### Description:

This query returns a list of top-selling products in a specific month.

### Usage:

Replace `'2024-12-01'` with a date within the desired month to calculate the total quantity sold for that month.

---

## Query 3: High-Value Customers for the Previous Month

```sql
SELECT
    c.first_name,
    SUM(o.total_amount) as total_amount
FROM
    customer c
JOIN
    "order" o
ON
    c.customer_id = o.customer_id
WHERE
    DATE_TRUNC('month', o.order_date) = DATE_TRUNC('month', CURRENT_DATE) - INTERVAL '1 month'
GROUP BY
    c.first_name
HAVING
    SUM(o.total_amount) > 500
```

### Description:

This query identifies customers who spent more than $500 in the previous month.

### Usage:

Run the query as-is to get results for the previous month relative to the current date.

---

## Query 4: Search for Products by Keyword

```sql
SELECT
    product_id,
    name
FROM
    product
WHERE
    name ILIKE '%camera%' OR description ILIKE '%camera%' -- Replace 'camera' with the desired keyword
```

### Description:

This query retrieves all products that contain a specific keyword in their name or description. The keyword search is case-insensitive.

### Usage:

Replace `%camera%` with `%<your_keyword>%` to search for products based on a specific term.

---

## Query 5: Recommend Products from the Same Category Excluding Purchased Products

### Query with Joins

```sql
SELECT
    p.id,
    p.name,
    p.category_id
FROM
    product p
WHERE
    p.category_id = 5 -- Replace with the desired category ID
    AND NOT EXISTS (
        SELECT 1
        FROM "order" o
        INNER JOIN order_details od ON o.id = od.order_id
        INNER JOIN product purchased ON od.product_id = purchased.id
        WHERE o.customer_id = 100 -- Replace with the current customer ID
          AND purchased.id = p.id
    )
```

### Query Using Our Denormalized Sales History Table

```sql
SELECT
    p.id,
    p.name,
    p.category_id
FROM
    product p
WHERE
    p.category_id = 5 -- Replace with the desired category ID
    AND NOT EXISTS (
        SELECT 1
        FROM sales_history sh
        WHERE sh.customer_id = 100 -- Replace with the current customer ID
          AND sh.product_id = p.id
    )
```

### Description:

This query recommends products from the same category that the customer has not purchased before. It can be achieved either by using joins or a denormalized [sales_history](./sales-history.md) table for better performance.

### Usage:

Replace `5` with the desired category ID and `100` with the current customer ID to fetch recommendations.

---

## Query 6: Products per Category

```sql
SELECT
    c.category_id,
    COUNT(p.product_id) as total_products
FROM
    category c
INNER JOIN
    product p ON c.category_id = p.category_id
GROUP BY
    c.category_id
ORDER BY c.category_id
```

### Description:

This query counts the total number of products in each category.

### Usage:

Run the query as-is to get a count of products per category.

---

## Query 7: Top Customers by Spending

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, '', c.last_name) as customer_name,
    SUM(o.total_amount) as total_sum
FROM
    "order" o
JOIN
    customer c ON c.customer_id = o.customer_id
GROUP BY
    c.customer_id
ORDER BY
    total_sum DESC
LIMIT 10
```

### Description:

This query identifies the top 10 customers by their total spending.

### Usage:

Execute the query to get the list of top spending customers.

---

## Query 8: Recent Orders with Customer Information

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) as customer_name,
    c.email,
    o.order_date
FROM
    "order" o
JOIN
    customer c ON c.customer_id = o.customer_id
ORDER BY
    order_date DESC
LIMIT 1000
```

### Description:

This query retrieves the 1000 most recent orders with customer details.

### Usage:

Run the query to get the latest orders and associated customer information.

---

## Query 9: Low Stock Products

```sql
SELECT
    product_id,
    stock_quantity
FROM
    product
WHERE
    stock_quantity < 10
```

### Description:

This query finds all products with stock quantity less than 10 units.

### Usage:

Execute the query to identify products that need restocking.

---

## Query 10: Category Revenue

```sql
SELECT
    p.category_id,
    SUM(sub.total_amount) as total_revenue
FROM
    product p
JOIN (
    SELECT
        od.product_id,
        o.total_amount
    FROM
        order_details od
    JOIN
        "order" o ON od.order_id = o.order_id
) sub ON p.product_id = sub.product_id
GROUP BY
    p.category_id
```

### Description:

This query calculates the total revenue generated by each product category.

### Usage:

Run the query to analyze revenue performance by category.

---

## Performance Optimizations

### Summary Table

| Query Description                | Original Execution Time | Optimization Technique             | Optimized Execution Time | Link to Details                                     |
| -------------------------------- | ----------------------- | ---------------------------------- | ------------------------ | --------------------------------------------------- |
| Products per Category            | 38.293 ms               | Aggregate before join              | 12.745 ms                | [View Details](#products-per-category-optimization) |
| Top Customers by Spending        | 1886.650 ms             | Materialized view + covering index | 164.752 ms               | [View Details](#top-customers-optimization)         |
| Recent Orders with Customer Info | 951.314 ms              | CTE for initial limiting           | 166.806 ms               | [View Details](#recent-orders-optimization)         |
| Low Stock Products               | 19.694 ms               | Expression index                   | 0.759 ms                 | [View Details](#low-stock-optimization)             |
| Category Revenue                 | 4170.749 ms             | Materialized view                  | 0.025 ms                 | [View Details](#category-revenue-optimization)      |

### Detailed Optimizations

#### Products per Category Optimization

**Problem:**

- The original query performs a JOIN operation before aggregation
- This leads to unnecessary data processing as we're joining more rows than needed
- No index on the category_id foreign key

**Solution Steps:**

1. Create an index on the foreign key:

```sql
CREATE INDEX idx_product_category ON product(category_id);
```

2. Restructure query to aggregate before joining:

```sql
SELECT
    c.category_id,
    p.product_count AS total_products
FROM
    category c
INNER JOIN (
    SELECT
        category_id,
        COUNT(*) as product_count
    FROM
        product
    GROUP BY
        category_id
) p ON c.category_id = p.category_id;
```

**Why It's Faster:**

- Reduces the number of rows being joined
- Uses index for efficient category lookups
- Aggregates data before joining, minimizing the data processed

#### Top Customers Optimization

**Problem:**

- Large table scans for aggregating order amounts
- Repeated calculations for frequently accessed data
- No covering index for the required columns

**Solution Steps:**

1. Create a covering index:

```sql
CREATE INDEX idx_order_customer_amount ON "order"(customer_id, total_amount);
```

2. Create a materialized view for pre-calculated totals:

```sql
CREATE MATERIALIZED VIEW customer_total_spending AS
SELECT
    customer_id,
    SUM(total_amount) AS total_spending
FROM "order"
GROUP BY customer_id;
```

3. Optimize query to use materialized view:

```sql
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    mv.total_spending
FROM
    customer c
JOIN
    customer_total_spending mv ON c.customer_id = mv.customer_id
ORDER BY
    mv.total_spending DESC
LIMIT 10;
```

**Maintenance Required:**

- Refresh materialized view periodically:

```sql
REFRESH MATERIALIZED VIEW customer_total_spending;
```

#### Recent Orders Optimization

**Problem:**

- Joining full tables before limiting results
- Sorting large dataset after join
- No index on order_date

**Solution Steps:**

1. Create index on order_date:

```sql
CREATE INDEX idx_order_date ON "order"(order_date DESC);
```

2. Use CTE to limit rows before joining:

```sql
WITH latest_orders AS (
    SELECT
        customer_id,
        order_date
    FROM
        "order"
    ORDER BY
        order_date DESC
    LIMIT 1000
)
SELECT
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.email,
    lo.order_date
FROM
    latest_orders lo
JOIN
    customer c ON c.customer_id = lo.customer_id
ORDER BY
    lo.order_date DESC;
```

**Why It's Faster:**

- Reduces the number of rows before joining
- Uses index for efficient sorting
- Minimizes memory usage for sorting operation

#### Low Stock Optimization

**Problem:**

- Full table scan for checking stock quantity
- No index support for the WHERE clause condition

**Solution Steps:**

1. Create an expression index:

```sql
CREATE INDEX idx_low_stock ON product(stock_quantity)
WHERE stock_quantity < 10;
```

2. Original query remains the same but uses index:

```sql
SELECT
    product_id,
    stock_quantity
FROM
    product
WHERE
    stock_quantity < 10;
```

**Why It's Faster:**

- Partial index only stores relevant rows
- Quick access to low stock products
- Smaller index size compared to full column index

#### Category Revenue Optimization

**Problem:**

- Complex joins with aggregation
- Repeated calculations of revenue
- No pre-aggregated data

**Solution Steps:**

1. Create materialized view for category sales:

```sql
CREATE MATERIALIZED VIEW product_sales AS
    SELECT
        p.category_id,
        SUM(o.total_amount) as total_revenue
    FROM
        order_details od
    JOIN
        "order" o ON od.order_id = o.order_id
    JOIN
        product p ON od.product_id = p.product_id
    GROUP BY
        p.category_id;
```

2. Simplified query using materialized view:

```sql
SELECT
    category_id,
    total_revenue
FROM
    product_sales;
```

**Maintenance Required:**

- Refresh the materialized view based on business needs:

```sql
REFRESH MATERIALIZED VIEW product_sales;
```
