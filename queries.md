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

## Query 2: Total Quantity Sold for a Specific Month

```sql
SELECT
    SUM(od.quantity) AS total_quantity_sold
FROM
    order_details od
JOIN
    "order" o ON od.order_id = o.order_id
WHERE
    DATE_TRUNC('month', o.order_date) = DATE_TRUNC('month', '2024-12-01'::DATE) -- Replace with the desired month
```

### Description:

This query calculates the total quantity of items sold during a specific month.

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
