# Sales, Purchasing, Inventory, and Management SQL Queries

## 1. Sales Department Queries

### a. Analyzing Sales Trends for Key Accounts
```sql
SELECT `Customer`, 
       SUM(`Total Price`) AS total_revenue, 
       COUNT(`Order ID`) AS total_orders
FROM sales
WHERE `Order Date` BETWEEN '2020-01-01' AND '2024-12-31'
GROUP BY `Customer`
ORDER BY total_revenue DESC
LIMIT 10;
```
*Helps target high-value customers for upselling and personalized marketing.*

### b. Monthly Sales Performance Breakdown
```sql
SELECT DATE_FORMAT(`Order Date`, '%Y-%m') AS sales_month, 
       SUM(`Total Price`) AS monthly_revenue
FROM sales
GROUP BY sales_month
ORDER BY sales_month DESC;
```
*Shows revenue trends by month/quarter.*

### c. Customer Order Frequency Analysis
```sql
SELECT `Customer`, 
       COUNT(`Order ID`) AS total_orders
FROM sales
GROUP BY `Customer`
HAVING total_orders > 5
ORDER BY total_orders DESC;
```
*Identifies repeat customers to retain relationships and improve the customer experience.*

### d. Identifying Slow-Moving Products
```sql
SELECT `Product`, 
       SUM(`Quantity`) AS total_sold
FROM sales
WHERE `Order Date` >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY `Product`
ORDER BY total_sold ASC
LIMIT 10;
```
*Finds least-selling products and adjusts inventory based on seasonal factors.*

## 2. Purchasing Department Queries

### a. Identifying Products with Frequent Reordering
```sql
SELECT p.`Name` AS product_name, 
       p.`Reorder Point`, 
       i.`Quantity` AS stock_quantity
FROM products p
JOIN inventory i ON p.`Product ID` = i.`Product ID`
WHERE i.`Quantity` < p.`Reorder Point`
ORDER BY i.`Quantity` ASC;
```
*Helps in inventory planning.*

### b. Average Cost per Supplier for Key Products
```sql
SELECT po.`Supplier ID`, 
       s.`Name` AS supplier_name, 
       p.`Name` AS product_name, 
       AVG(po.`Total Cost`) AS avg_cost
FROM purchase_orders po
JOIN suppliers s ON po.`Supplier ID` = s.`Supplier ID`
JOIN products p ON po.`Purchase ID` = p.`Product ID`
WHERE p.`Name` IN ('D-Calcium Pantothenate (VB5)', 'Vitamin C,Na,Ca,DC', 'Berberine Hcl')
GROUP BY po.`Supplier ID`, s.`Name`, p.`Name`
ORDER BY avg_cost ASC;
```
*Finds the cheapest supplier for each product to reduce costs.*

## 3. Inventory Management Queries

### a. Demand Forecast vs. Current Inventory vs. Restock Level
```sql
SELECT p.Name AS product_name, i.Quantity AS stock_quantity, p.Reorder Point 
FROM inventory_df i 
JOIN products_df p ON i.Product ID = p.Product ID 
WHERE i.Quantity < p.Reorder Point 
ORDER BY i.Quantity ASC;
```
```sql
SELECT i.Product ID, p.Name AS product_name, i.Quantity AS stock_quantity, i.Demand Forecast 
FROM inventory_df i 
JOIN products_df p ON i.Product ID = p.Product ID 
ORDER BY i.Demand Forecast DESC;
```

### b. Restock Level Formula

Restock Amount=(Forecasted Sales×Lead Time)+Safety Stock−Current Inventory
```sql
SELECT p.`Name` AS product_name, i.`Quantity` AS current_stock, i.`Demand Forecast` AS forecasted_sales, s.`Lead Time (days)`, 
       (i.`Demand Forecast` * s.`Lead Time (days)`) + 100 AS recommended_restock_amount 
FROM inventory i 
JOIN products p ON i.`Product ID` = p.`Product ID` 
JOIN `suppliers_df` s ON p.`Product ID` = s.`Supplier ID` 
ORDER BY recommended_restock_amount DESC;
```

## 4. Management Team Queries

### a. Profit Margin Analysis by Product
```sql
SELECT Product, 
       SUM('Total Price') AS revenue, 
       SUM(Quantity * 'Unit Price') AS cost, 
       (SUM('Total Price') - SUM(Quantity * 'Unit Price')) AS profit, 
       ((SUM('Total Price') - SUM(Quantity * 'Unit Price')) / SUM('Total Price')) * 100 AS profit_margin
FROM sales 
GROUP BY Product
ORDER BY profit_margin DESC;
```
*Identifies the profitability of each product.*

### b. Revenue Contribution by Industry Type
```sql
SELECT 'Industry Type', 
       SUM('Total Price') AS total_revenue
FROM sales s
JOIN customers c ON s.Customer = c.Name
GROUP BY 'Industry Type'
ORDER BY total_revenue DESC;
```
*Identifies the most profitable industries for targeted marketing.*
