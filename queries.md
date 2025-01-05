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
