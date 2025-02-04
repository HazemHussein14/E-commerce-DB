# E-commerce Database Data Generation and Import Guide

## Prerequisites

- Python 3.x
- PostgreSQL installed
- Python packages: pandas, numpy, faker

## Step 1: Generate Sample Data

1. Create a directory for your project
2. Save the following Python script as `data_generator.py`:

   ```python
   import pandas as pd
   import numpy as np
   from faker import Faker
   from datetime import datetime, timedelta
   import random
   import os

   # Initialize Faker
   fake = Faker()
   Faker.seed(12345)
   np.random.seed(12345)

   class EcommerceDataGenerator:
       def __init__(self, output_dir='data'):
           self.output_dir = output_dir
           os.makedirs(output_dir, exist_ok=True)

       def generate_categories(self, num_categories=100):
        print("Generating categories...")
        main_categories = [
            'Electronics', 'Clothing', 'Home & Garden', 'Sports & Outdoors',
            'Beauty & Personal Care', 'Toys & Games', 'Automotive', 'Health & Wellness',
            'Food & Beverages', 'Office Supplies', 'Pet Supplies', 'Tools & Home Improvement',
            'Jewelry & Watches', 'Arts & Crafts', 'Baby & Kids', 'Music & Instruments'
        ]

        categories = []
        for category_id, category_name in enumerate(main_categories, 1):
            categories.append({
                'category_id': category_id,
                'category_name': category_name
            })

        df_categories = pd.DataFrame(categories)
        df_categories.to_csv(f'{self.output_dir}/categories.csv', index=False)
        return df_categories

       def generate_products(self, num_products=100_000, categories_df=None):
           print("Generating products...")
           brands = [
               'TechPro', 'StyleLife', 'HomeEssentials', 'SportMaster', 'BeautyGlow',
               'GamerX', 'AutoMax', 'HealthPlus', 'FoodFresh', 'OfficePro'
           ]

           products = []
           for i in range(num_products):
               category_id = np.random.choice(categories_df['category_id'])
               brand = np.random.choice(brands)
               price = np.random.choice([
                   np.random.uniform(5, 100),    # 60% cheap items
                   np.random.uniform(100, 500),   # 30% mid-range items
                   np.random.uniform(500, 3000)   # 10% expensive items
               ], p=[0.6, 0.3, 0.1])

               products.append({
                   'product_id': i + 1,
                   'category_id': category_id,
                   'name': f"{brand} {fake.word().title()} {fake.word().title()}",
                   'author': brand,  # Using author field as brand
                   'description': fake.text(max_nb_chars=200),
                   'price': round(price, 2),
                   'stock_quantity': np.random.randint(0, 1000)
               })

               if (i + 1) % 10000 == 0:
                   print(f"Generated {i + 1} products")

           df_products = pd.DataFrame(products)
           df_products.to_csv(f'{self.output_dir}/products.csv', index=False)
           return df_products

       def generate_customers(self, num_customers=1_000_000):
           print("Generating customers...")
           customers = []

           for i in range(num_customers):
               customers.append({
                   'customer_id': i + 1,
                   'first_name': fake.first_name(),
                   'last_name': fake.last_name(),
                   'email': fake.unique.email(),
                   'password': fake.sha256()
               })

               if (i + 1) % 100000 == 0:
                   print(f"Generated {i + 1} customers")

           df_customers = pd.DataFrame(customers)
           df_customers.to_csv(f'{self.output_dir}/customers.csv', index=False)
           return df_customers

       def generate_orders(self, num_orders=2_000_000, products_df=None, customers_df=None):
           print("Generating orders and order details...")
           orders = []
           order_details = []
           order_detail_id = 1

           start_date = datetime(2023, 1, 1)
           end_date = datetime(2024, 12, 31)

           for i in range(num_orders):
               customer_id = np.random.choice(customers_df['customer_id'])
               order_date = fake.date_time_between(start_date=start_date, end_date=end_date)

               # Create order
               order_id = i + 1
               total_amount = 0

               # Generate 1-8 items per order
               num_items = np.random.randint(1, 9)

               # Generate order details
               order_products = products_df.sample(n=num_items)
               for _, product in order_products.iterrows():
                   quantity = np.random.randint(1, 11)
                   unit_price = product['price']
                   total_amount += quantity * unit_price

                   order_details.append({
                       'order_detail_id': order_detail_id,
                       'order_id': order_id,
                       'product_id': product['product_id'],
                       'quantity': quantity,
                       'unit_price': unit_price
                   })
                   order_detail_id += 1

               orders.append({
                   'order_id': order_id,
                   'customer_id': customer_id,
                   'order_date': order_date,
                   'total_amount': round(total_amount, 2)
               })

               if (i + 1) % 100000 == 0:
                   print(f"Generated {i + 1} orders")

           # Save orders
           df_orders = pd.DataFrame(orders)
           df_orders.to_csv(f'{self.output_dir}/orders.csv', index=False)

           # Save order details
           df_order_details = pd.DataFrame(order_details)
           df_order_details.to_csv(f'{self.output_dir}/order_details.csv', index=False)

           return df_orders, df_order_details

   def generate_all_data():
       generator = EcommerceDataGenerator()

       # Generate all data
       categories_df = generator.generate_categories()
       products_df = generator.generate_products(categories_df=categories_df)
       customers_df = generator.generate_customers()
       generator.generate_orders(products_df=products_df, customers_df=customers_df)

   if __name__ == "__main__":
       generate_all_data()
   ```

3. Create a `data` directory in your project folder
4. Run the data generator:
   ```bash
   python data_generator.py
   ```
   This will generate CSV files in the `data` directory:
   - categories.csv (100 rows)
   - products.csv (100,000 rows)
   - customers.csv (1,000,000 rows)
   - orders.csv (2,000,000 rows)
   - order_details.csv (~9,000,000 rows)

## Step 2: Create Database Tables

1. Create your database if not exists:

   ```sql
   CREATE DATABASE your_database_name;
   ```

2. Create the tables using the schema:

   ```sql
   CREATE TABLE category (
       category_id SERIAL PRIMARY KEY,
       category_name VARCHAR(100) NOT NULL
   );

   CREATE TABLE product (
       product_id SERIAL PRIMARY KEY,
       category_id INT REFERENCES category(category_id),
       name VARCHAR(100) NOT NULL,
       author VARCHAR(100) NOT NULL,
       description TEXT,
       price NUMERIC(10, 2) NOT NULL,
       stock_quantity INT NOT NULL
   );

   CREATE TABLE customer (
       customer_id SERIAL PRIMARY KEY,
       first_name VARCHAR(50) NOT NULL,
       last_name VARCHAR(50) NOT NULL,
       email VARCHAR(100) UNIQUE NOT NULL,
       password VARCHAR(255) NOT NULL
   );

   CREATE TABLE "order" (
       order_id SERIAL PRIMARY KEY,
       customer_id INT REFERENCES customer(customer_id),
       order_date TIMESTAMP NOT NULL DEFAULT NOW(),
       total_amount NUMERIC(10, 2) NOT NULL
   );

   CREATE TABLE order_details (
       order_detail_id SERIAL PRIMARY KEY,
       order_id INT REFERENCES "order"(order_id),
       product_id INT REFERENCES product(product_id),
       quantity INT NOT NULL,
       unit_price NUMERIC(10, 2) NOT NULL
   );
   ```

## Step 3: Import Data

1. Create `load_data.sql` with the following content:

   ```sql
   -- Disable triggers
   ALTER TABLE category DISABLE TRIGGER ALL;
   ALTER TABLE product DISABLE TRIGGER ALL;
   ALTER TABLE customer DISABLE TRIGGER ALL;
   ALTER TABLE "order" DISABLE TRIGGER ALL;
   ALTER TABLE order_details DISABLE TRIGGER ALL;

   -- Clear existing data
   TRUNCATE category, product, customer, "order", order_details RESTART IDENTITY CASCADE;

   -- Load data using COPY
   \COPY category FROM 'data/categories.csv' WITH (FORMAT csv, HEADER true);
   \COPY product FROM 'data/products.csv' WITH (FORMAT csv, HEADER true);
   \COPY customer FROM 'data/customers.csv' WITH (FORMAT csv, HEADER true);
   \COPY "order" FROM 'data/orders.csv' WITH (FORMAT csv, HEADER true);
   \COPY order_details FROM 'data/order_details.csv' WITH (FORMAT csv, HEADER true);

   -- Re-enable triggers
   ALTER TABLE category ENABLE TRIGGER ALL;
   ALTER TABLE product ENABLE TRIGGER ALL;
   ALTER TABLE customer ENABLE TRIGGER ALL;
   ALTER TABLE "order" ENABLE TRIGGER ALL;
   ALTER TABLE order_details ENABLE TRIGGER ALL;
   ```

2. Run the import script:
   ```bash
   psql -d your_database_name -f load_data.sql
   ```

## Directory Structure

```
your_project/
├── data_generator.py
├── load_data.sql
└── data/
    ├── categories.csv
    ├── products.csv
    ├── customers.csv
    ├── orders.csv
    └── order_details.csv
```

## Note on Execution Time

- Data generation: ~8-10 minutes
- Data import: ~5-8 minutes
- Total process time: ~15-20 minutes
