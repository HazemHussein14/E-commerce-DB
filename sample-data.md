### Insert Sample Data
After creating the tables, populate them with the following sample data to test functionality:

```sql
-- Insert Categories
INSERT INTO category (category_name) VALUES
    ('Electronics'),
    ('Books'),
    ('Clothing');

-- Insert Products
INSERT INTO product (category_id, "name", description, price, stock_quantity) VALUES
    (1, 'Smartphone', 'Latest model smartphone with 128GB storage', 699.99, 50),
    (1, 'Laptop', 'High-performance laptop for gaming and work', 1299.99, 20),
    (2, 'Novel', 'A best-selling fiction book', 15.99, 100),
    (3, 'T-Shirt', 'Comfortable cotton t-shirt', 9.99, 200);

-- Insert Customers
INSERT INTO customer (first_name, last_name, email, "password") VALUES
    ('John', 'Doe', 'john.doe@example.com', 'hashed_password_1'),
    ('Jane', 'Smith', 'jane.smith@example.com', 'hashed_password_2');

-- Insert Orders
INSERT INTO "order" (customer_id, order_date, total_amount) VALUES
    (1, '2024-12-01 10:00:00', 729.97),
    (2, '2024-12-02 14:30:00', 25.98);

-- Insert Order Details
INSERT INTO order_details (order_id, product_id, quantity, unit_price) VALUES
    (1, 1, 1, 699.99),
    (1, 4, 3, 9.99),
    (2, 3, 2, 15.99);
```
