# Sales History Table Documentation

## Overview

The `Sales History` table is a denormalized representation of sales data within the e-commerce database. It aggregates data from multiple normalized tables (`Order`, `Order Details`, `Product`, `Customer`, and `Category`) into a single table to optimize data retrieval for reporting and analytics.

This table is designed to minimize complex joins and improve performance for querying historical sales data.

---

## Table Structure

### Columns

The `Sales History` table includes the following columns:

- **sale_id**: Unique identifier for each sale (Primary Key).
- **order_id**: Reference to the original order.
- **customer_id**: Reference to the customer who placed the order.
- **customer_name**: Full name of the customer (concatenation of `first_name` and `last_name`).
- **email**: Email address of the customer.
- **order_date**: Date and time of the sale.
- **category_name**: Name of the product category.
- **product_id**: Reference to the product.
- **product_name**: Name of the product.
- **quantity**: Number of units sold.
- **unit_price**: Price per unit of the product.
- **total_amount**: Total amount for the sale (calculated as `quantity * unit_price`).

---

## Purpose

The purpose of this table is to provide a single, query-friendly source for analyzing sales history. Typical use cases include:

1. Generating reports for sales performance by customer, product, or category.
2. Identifying trends in sales over time.
3. Analyzing high-value customers or top-selling products.

By consolidating sales data into a single table, the `Sales History` table reduces the complexity of retrieving and aggregating data for business intelligence purposes.

---

## Table Creation Script

The following script creates the `Sales History` table and demonstrates how data is denormalized into it:

```sql
CREATE TABLE sales_history (
    sale_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    customer_id INT NOT NULL,
    customer_name VARCHAR(150) NOT NULL,
    email VARCHAR(100) NOT NULL,
    order_date TIMESTAMP NOT NULL,
    category_name VARCHAR(100) NOT NULL,
    product_id INT NOT NULL,
    product_name VARCHAR(100) NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10, 2) NOT NULL CHECK (unit_price >= 0),
    total_amount NUMERIC(10, 2) NOT NULL CHECK (total_amount >= 0),
);
```

---

## Data Insertion Logic

To populate the `Sales History` table, you can use the following query, which joins data from the relevant normalized tables (assuming we don't have a huge amount of data in our normalized tables):

```sql
INSERT INTO sales_history (
    order_id, customer_id, customer_name, email, order_date,
    category_name, product_id, product_name, quantity, unit_price, total_amount
)
SELECT
    o.order_id,
    c.customer_id,
    CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
    c.email,
    o.order_date,
    cat.category_name,
    p.product_id,
    p.name AS product_name,
    od.quantity,
    od.unit_price,
    (od.quantity * od.unit_price) AS total_amount
FROM
    "order" o
JOIN
    customer c ON o.customer_id = c.customer_id
JOIN
    order_details od ON o.order_id = od.order_id
JOIN
    product p ON od.product_id = p.product_id
JOIN
    category cat ON p.category_id = cat.category_id;
```

---

## Note

We will create a trigger to automatically update the Sales History table whenever changes occur in the source tables. This will ensure the denormalized data remains accurate and up-to-date.
