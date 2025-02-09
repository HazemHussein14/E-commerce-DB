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
    - [Total Revenue for a Specific Date](#total-revenue-for-a-specific-date-optimization)
    - [Top Selling Products Optimization](#top-selling-products-optimization)
    - [Monthly High-Spending Customers Optimization](#monthly-high-spending-customers-optimization)
    - [Product Search Optimization](#product-search-optimization)
    - [Personalized Product Recommendations Optimization](#personalized-product-recommendations-optimization)
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
    DATE_TRUNC('month', o.order_date) =  '2024-12-01' -- Replace with the specific date
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

## Query 5: Recommend Products from the Same Category and Author Excluding Purchased Products

### Query with Joins

```sql
SELECT
    p.product_id,
    p.name,
    p.author,
    p.description,
    p.price,
    p.stock_quantity
FROM
    product p
JOIN
    category c ON p.category_id = c.category_id
WHERE
    p.category_id IN (
        SELECT
            p2.category_id
        FROM
            product p2
        JOIN
            order_details od ON p2.product_id = od.product_id
        JOIN
            order o ON od.order_id = o.order_id
        WHERE
            o.customer_id = ? -- Replace with the specific customer ID
    )
    AND p.author IN (
        SELECT
            p2.author
        FROM
            product p2
        JOIN
            order_details od ON p2.product_id = od.product_id
        JOIN
            order o ON od.order_id = o.order_id
        WHERE
            o.customer_id = ? -- Replace with the specific customer ID
    )
    AND p.product_id NOT IN (
        SELECT
            od.product_id
        FROM
            order_details od
        JOIN
            order o ON od.order_id = o.order_id
        WHERE
            o.customer_id = ? -- Replace with the specific customer ID
    );
```

### Query Using Our Denormalized Sales History Table

```sql
SELECT
    p.product_id,
    p.name,
    p.category_id
FROM
    product p
WHERE p.category_id IN (
    SELECT category_id
    FROM product
    WHERE product_id IN (
        SELECT product_id
        FROM sales_history sh
        WHERE customer_id = 20
    )
)
AND p.author IN (
    SELECT author
    FROM product
    WHERE product_id IN (
        SELECT product_id
        FROM sales_history sh
        WHERE customer_id = 20
    )
)
AND p.product_id NOT IN (
    SELECT product_id
    FROM sales_history sh
    WHERE customer_id = 20
);
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

| Query Description                    | Original Execution Time | Optimization Technique                | Optimized Execution Time | Link to Details                                                    |
| ------------------------------------ | ----------------------- | ------------------------------------- | ------------------------ | ------------------------------------------------------------------ |
| Total Revenue for a Specific Date    | 130.877 ms              | Index on DATE(order_date)             | 2.685 ms                 | [View Details](#total-revenue-for-a-specific-date-optimization)    |
| Top Selling Products                 | 1908.878 ms             | Covering index + range date filter    | 691.440 ms               | [View Details](#top-selling-products-optimization)                 |
| Monthly High-Spending Customers      | 1008.952 ms             | Index on order_date + range filtering | 129.400 ms               | [View Details](#monthly-high-spending-customers-optimization)      |
| Product Search Optimization          | 1116.505 ms             | Full-text search with GIN index       | 2.423 ms                 | [View Details](#product-search-optimization)                       |
| Personalized Product Recommendations | 2784.692 ms             | Covering index                        | 7.355 ms                 | [View Details](#personalized-product-recommendations-optimization) |
| Products per Category                | 38.293 ms               | Aggregate before join                 | 12.745 ms                | [View Details](#products-per-category-optimization)                |
| Top Customers by Spending            | 1886.650 ms             | Materialized view + covering index    | 164.752 ms               | [View Details](#top-customers-optimization)                        |
| Recent Orders with Customer Info     | 951.314 ms              | CTE for initial limiting              | 166.806 ms               | [View Details](#recent-orders-optimization)                        |
| Low Stock Products                   | 19.694 ms               | Expression index                      | 0.759 ms                 | [View Details](#low-stock-optimization)                            |
| Category Revenue                     | 4170.749 ms             | Materialized view                     | 0.025 ms                 | [View Details](#category-revenue-optimization)                     |

### Detailed Optimizations

---

#### Total Revenue for a Specific Date Optimization

**Problem:**

- The original query performs a full table scan to filter orders by date.
- No index on the `order_date` column for date-specific filtering.

**Solution Steps:**

1. Create an index on the expression `DATE(order_date)`:

```sql
CREATE INDEX idx_order_date ON "order" ((DATE(order_date)));
```

2. Optimized query:

```sql
SELECT
    SUM(total_amount) AS total_revenue
FROM
    "order"
WHERE
    DATE(order_date) = '2024-12-01'; -- Replace with the specific date
```

**Why It's Faster:**

- The index allows PostgreSQL to quickly locate rows matching the specific date.
- Reduces the number of rows scanned during query execution.

**Execution Time:**

- Before Optimization: 130.877 ms
- After Optimization: 2.685 ms

---

#### Top Selling Products Optimization

**Problem:**

- The original query uses `DATE_TRUNC` for filtering, which is computationally expensive.
- Joins and aggregations are performed on a large dataset.

**Solution Steps:**

1. Replace `DATE_TRUNC` with a range filter:

```sql
WHERE
    o.order_date >= '2024-12-01'
    AND o.order_date < '2025-01-01'
```

2. Create a covering index for `order_details`:

```sql
CREATE INDEX idx_orderdetails_covering ON order_details(order_id, product_id, quantity);
```

3. Optimized query:

```sql
SELECT
    od.product_id,
    SUM(od.quantity) AS total_quantity_sold
FROM
    order_details od
JOIN
    "order" o ON od.order_id = o.order_id
WHERE
    o.order_date >= '2024-12-01'
    AND o.order_date < '2025-01-01'
GROUP BY
    od.product_id
ORDER BY
    total_quantity_sold DESC
LIMIT 10;
```

**Why It's Faster:**

- Range filtering is more efficient than `DATE_TRUNC`.
- The covering index reduces the need for additional table lookups.

**Execution Time:**

- Before Optimization: 1908.878 ms
- After Optimization: 691.440 ms

---

#### Monthly High-Spending Customers Optimization

**Problem:**

- The original query uses `DATE_TRUNC` and joins the `customer` table unnecessarily.
- No index on `order_date` for efficient filtering.

**Solution Steps:**

1. Create an index on `order_date`:

```sql
CREATE INDEX idx_order_date ON "order"(order_date);
```

2. Optimize the query to use range filtering and remove unnecessary joins:

```sql
SELECT
    customer_id,
    SUM(total_amount) AS total_amount
FROM
    "order"
WHERE
    order_date >= DATE_TRUNC('month', CURRENT_DATE) - INTERVAL '2 month'
    AND order_date < DATE_TRUNC('month', CURRENT_DATE)
GROUP BY
    customer_id
HAVING
    SUM(total_amount) > 500;
```

**Why It's Faster:**

- Range filtering is more efficient than `DATE_TRUNC`.
- Removing unnecessary joins reduces query complexity.

**Execution Time:**

- Before Optimization: 1008.952 ms
- After Optimization: 129.400 ms

---

#### Product Search Optimization

**Problem:**

- The original query uses `ILIKE`, which performs a full table scan and is case-insensitive.
- No index support for `ILIKE` with wildcard patterns.

**Solution Steps:**

1. Add a `tsvector` column for full-text search:

```sql
ALTER TABLE product ADD COLUMN search_text_vector tsvector;
```

2. Populate the `tsvector` column:

```sql
UPDATE product
SET search_text_vector = to_tsvector('english', name || ' ' || COALESCE(description, ''));
```

3. Create a GIN index for fast searching:

```sql
CREATE INDEX idx_product_search ON product USING gin(search_text_vector);
```

4. Optimized query:

```sql
SELECT
    product_id,
    name,
    description
FROM
    product
WHERE
    search_text_vector @@ plainto_tsquery('camera');
```

**Why It's Faster:**

- Full-text search is optimized for pattern matching.
- The GIN index allows for fast lookups.

**Execution Time:**

- Before Optimization: 1116.505 ms
- After Optimization: 2.423 ms

---

#### Personalized Product Recommendations Optimization

**Problem:**

- The original query uses multiple subqueries with `IN` clauses, which are inefficient.
- No covering index for the `sales_history` table.

**Solution Steps:**

1. Create a covering index for `sales_history`:

```sql
CREATE INDEX idx_covering_fkeys ON sales_history(customer_id, product_id);
```

2. Use the same query:

```sql
SELECT
    p.product_id,
    p.name,
    p.category_id
FROM
    product p
WHERE p.category_id IN (
    SELECT category_id
    FROM product
    WHERE product_id IN (
        SELECT product_id
        FROM sales_history sh
        WHERE customer_id = 20
    )
)
AND p.author IN (
    SELECT author
    FROM product
    WHERE product_id IN (
        SELECT product_id
        FROM sales_history sh
        WHERE customer_id = 20
    )
)
AND p.product_id NOT IN (
    SELECT product_id
    FROM sales_history sh
    WHERE customer_id = 20
);
```

**Why It's Faster:**

- The covering index reduces the need for additional table lookups.

**Execution Time:**

- Before Optimization: 2784.692 ms
- After Optimization: 7.355 ms

**Another Solution Using CTE**

1. Create a covering index for `sales_history`:
2. Create a covering index for `product`

```sql
CREATE INDEX idx_sales_covering ON sales_history(customer_id, product_id);
CREATE INDEX idx_product_covering ON sales_history(category_id, author);
```

3. Use CTE instead of repeating sub-queries
4. Use `EXISTS` instead of `IN`

```sql
WITH customer_products AS (
    SELECT p.product_id, p.category_id, p.author
    FROM product p
    WHERE EXISTS (
        SELECT 1
        FROM sales_history sh
        WHERE sh.customer_id = 20 AND sh.product_id = p.product_id
    )
)
SELECT p.product_id, p.name, p.category_id
FROM product p
WHERE p.category_id IN (SELECT category_id FROM customer_products)
AND p.author IN (SELECT author FROM customer_products)
AND NOT EXISTS (
    SELECT 1
    FROM sales_history sh
    WHERE sh.customer_id = 20 AND sh.product_id = p.product_id
);
```

**Benefits of this approach:**

- The covering indexes reduces the need for additional table lookups.
- The CTE simplifies the query and increase readability maintainability
- Using `EXISTS` stop searching as soon as it find a match

**Execution Time:**

- Before Optimization: 2784.692 ms
- After Optimization: 25.050 ms

---

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

---

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

---

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

---

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

---

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
