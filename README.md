# E-commerce Database

## Table of Contents

- [Overview](#overview)
- [Data Model](#data-model)
- [Database Schema and Setup](#database-schema-and-setup)
- [Denormalized Categories](./denormalized-categories.md)
- [Insights](#insights)
- [SQL Queries](#sql-queries)
- [Performance Optimization](#performance-optimization)

---

## Overview

This project provides a structured **e-commerce database** to manage products, customers, orders, and sales history efficiently. The database is designed for scalability and optimized for reporting and analytics.

---

## Data Model

The schema is designed to reflect real-world e-commerce operations, ensuring **data integrity** and **efficiency**. Below is the **Entity-Relationship Diagram (ERD):**

![ER Diagram](./ecommerce-ERD.drawio.png)

---

## Database Schema and Setup

Each table in the database plays a crucial role in maintaining the integrity and performance of the e-commerce system.  
Follow the **[setup guide](./setup.md)** to initialize the database and populate it with mock data.

---

## Insights

Below is an overview of the database tables and their **current row counts**:

| Table Name      | Number of Rows |
| --------------- | -------------- |
| `categories`    | 1,000,000      |
| `products`      | 100,000        |
| `customers`     | 2,000,000      |
| `orders`        | 12,850         |
| `order_details` | 8,998,652      |
| `sales_history` | 8,998,652      |

---

## SQL Queries

Commonly used SQL queries are available in the **[queries file](./queries.md)** for ease of reference. These queries help generate reports and perform data analysis on the e-commerce database.

### Included Queries:

1. **Total Revenue for a Specific Date**
2. **Monthly Top-Selling Products**
3. **High-Value Customers for the Previous Month**
4. **Search for Products by Keyword**
5. **Recommend Products from the Same Category**
6. **Products per Category**
7. **Top Customers by Spending**
8. **Recent Orders with Customer Information**
9. **Low Stock Products**
10. **Category Revenue**

---

## Performance Optimization

The database has been optimized to **enhance query execution times**. Below is a comparison of **before and after optimization**:

| Query Description                    | Original Execution Time | Optimized Execution Time |
| ------------------------------------ | ----------------------- | ------------------------ |
| Total Revenue for a Specific Date    | 130.877 ms              | 2.685 ms                 |
| Top Selling Products                 | 1908.878 ms             | 691.440 ms               |
| Monthly High-Spending Customers      | 1008.952 ms             | 129.400 ms               |
| Product Search Optimization          | 1116.505 ms             | 2.423 ms                 |
| Personalized Product Recommendations | 2784.692 ms             | 7.355 ms                 |
| Products per Category                | 38.293 ms               | 12.745 ms                |
| Top Customers by Spending            | 1886.650 ms             | 164.752 ms               |
| Recent Orders with Customer Info     | 951.314 ms              | 166.806 ms               |
| Low Stock Products                   | 19.694 ms               | 0.759 ms                 |
| Category Revenue                     | 4170.749 ms             | 0.025 ms                 |
