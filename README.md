
# Supermarket Management Database

## Overview

This repository contains the schema and SQL scripts for managing a customer and product database in a supermarket. The database handles customer information, product inventories, and transaction details.

## Schema

The database consists of four main tables:

1. **Customer**: Stores customer information.
2. **Product**: Stores product details.
3. **Transaction**: Stores transaction records.
4. **TransDetail**: Stores detailed information about each product in a transaction.

### Customer Table

```sql
CREATE TABLE Customer (
    cus_id INT AUTO_INCREMENT PRIMARY KEY,
    mob_num VARCHAR(12) UNIQUE NOT NULL,
    cus_name VARCHAR(30) NOT NULL,
    visits INT DEFAULT 0,
    membership ENUM('GOLD', 'SILVER', 'PLATINUM') DEFAULT 'GOLD'
);
```

### Product Table

```sql
CREATE TABLE Product (
    prod_id INT AUTO_INCREMENT PRIMARY KEY,
    rfid INT UNIQUE NOT NULL,
    prod_name VARCHAR(20) NOT NULL,
    qty_unit VARCHAR(10),
    unit_cost DECIMAL(10, 2) NOT NULL,
    stock_qty INT DEFAULT 0
);
```

### Transaction Table

```sql
CREATE TABLE Transaction (
    trans_id INT AUTO_INCREMENT PRIMARY KEY,
    cus_id INT,
    trans_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (cus_id) REFERENCES Customer(cus_id)
);
```

### TransDetail Table

```sql
CREATE TABLE TransDetail (
    detail_id INT AUTO_INCREMENT PRIMARY KEY,
    trans_id INT,
    prod_id INT,
    qty INT NOT NULL,
    total_cost DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (trans_id) REFERENCES Transaction(trans_id),
    FOREIGN KEY (prod_id) REFERENCES Product(prod_id)
);
```

## Sample Data Insertion

### Inserting Sample Customers

```sql
INSERT INTO Customer (mob_num, cus_name, visits, membership) 
VALUES 
('1234567890', 'Rakesh', 3, 'GOLD'),
('0987654321', 'Raju', 7, 'SILVER'),
('1122334455', 'Aarti', 12, 'PLATINUM'),
('2233445566', 'Rajesh', 1, 'GOLD'),
('4455667788', 'Amrita', 8, 'SILVER'),
('5566778899', 'Neha', 10, 'PLATINUM');
```

### Inserting Sample Products

```sql
INSERT INTO Product (rfid, prod_name, qty_unit, unit_cost, stock_qty)
VALUES 
(1001, 'Milk', 'Liter', 20.99, 100),
(1002, 'Coke', 'Can', 15.49, 200),
(1003, 'Bread', 'Loaf', 25.99, 150),
(1004, 'Eggs', 'Dozen', 30.49, 50),
(1005, 'Butter', 'Pack', 40.99, 75),
(1006, 'Cheese', 'Block', 60.99, 60),
(1007, 'Juice', 'Bottle', 18.99, 120);
```

### Inserting Sample Transactions and Details

```sql
-- Insert transactions for multiple customers
INSERT INTO Transaction (cus_id) VALUES (1), (2), (3);

-- Get the last inserted transaction IDs
SET @trans_id1 = LAST_INSERT_ID();
INSERT INTO Transaction (cus_id) VALUES (4);
SET @trans_id2 = LAST_INSERT_ID();
INSERT INTO Transaction (cus_id) VALUES (5);
SET @trans_id3 = LAST_INSERT_ID();

-- Insert transaction details for Rakesh's transaction
INSERT INTO TransDetail (trans_id, prod_id, qty, total_cost)
VALUES 
(@trans_id1, 1001, 2, 41.98),
(@trans_id1, 1003, 1, 25.99);

-- Insert transaction details for Raju's transaction
INSERT INTO TransDetail (trans_id, prod_id, qty, total_cost)
VALUES 
(@trans_id2, 1002, 3, 46.47),
(@trans_id2, 1005, 1, 40.99);

-- Insert transaction details for Aarti's transaction
INSERT INTO TransDetail (trans_id, prod_id, qty, total_cost)
VALUES 
(@trans_id3, 1004, 2, 60.98),
(@trans_id3, 1006, 1, 60.99);
```

## Operations

### Adding New Customers

To add a new customer:

```sql
INSERT INTO Customer (mob_num, cus_name, visits, membership) 
VALUES ('5556667777', 'John', 0, 'GOLD');
```

### Updating Customer Visits

To update the number of visits for a customer:

```sql
UPDATE Customer
SET visits = visits + ?
WHERE cus_id = ?;
```

Replace `?` with the number of visits to add and the customer ID.

### Printing All Products

To view all products in the inventory:

```sql
SELECT * FROM Product;
```

### Viewing Customer Details

To view details for a specific customer:

```sql
SELECT * FROM Customer
WHERE cus_id = ?;
```

Replace `?` with the customer ID.

### Calculating Membership Tier

To update the membership tier based on the number of visits:

```sql
UPDATE Customer
SET membership = CASE
    WHEN visits < 5 THEN 'GOLD'
    WHEN visits >= 5 AND visits < 10 THEN 'SILVER'
    ELSE 'PLATINUM'
END;
```

### Deleting a Product

To delete a product from the inventory:

```sql
DELETE FROM Product
WHERE prod_id = ?;
```

Replace `?` with the product ID.

### Analyzing Purchases of Good Regular Customers

To analyze the purchases of good regular customers, you can join the `Customer`, `Transaction`, and `TransDetail` tables:

```sql
SELECT 
    c.cus_name,
    p.prod_name,
    SUM(td.qty) AS total_quantity_purchased,
    SUM(td.total_cost) AS total_amount_spent
FROM 
    Customer c
JOIN 
    Transaction t ON c.cus_id = t.cus_id
JOIN 
    TransDetail td ON t.trans_id = td.trans_id
JOIN 
    Product p ON td.prod_id = p.prod_id
WHERE 
    c.visits > ?  -- Set the threshold for good regular customers
GROUP BY 
    c.cus_name, p.prod_name
ORDER BY 
    c.cus_name, total_amount_spent DESC;
```

Replace `?` with the minimum number of visits to consider a customer as a regular customer.

