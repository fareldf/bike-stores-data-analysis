# Bike Stores Data Analysis
Retail data holds powerful clues about business growth. By analyzing sales, productions, staff performance, and customers behavior from bike stores, this project uncovers actionable insights to support smarter decisions in marketing, expansion, and product strategy.

**Disclaimer**: this analysis is still at a **basic level** and may contain errors or inaccuracies. So, further validation are recommended.

## Dataset
**Bike Stores Dataset**

This is a sample database from [www.sqlservertutorial.net](https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.zip) for a retail bike store.

The sample database is copyrighted and cannot be used for commercial purposes. For example, it cannot be used for the following but is not limited to the purposes:
- Selling
- Including in paid courses

This dataset consists of these tables:
- brands: the brand’s information of bikes such as brand_id and brand_name.
- categories: the bike’s categories such as category_id and category_name.
- customers: the customer’s information including first name, last name, phone, email, street, city, state and zip code.
- order_items: the line items of a sales order. Each line item belongs to a sales order specified by the order_id column. A sales order line item includes product, order quantity, list price, and discount.
- orders: the sales order’s header information including customer, order status, order date, required date, shipped date. It also stores the information on where the sales transaction was created (store) and who created it (staff). Each sales order has a row in the sales_orders table. A sales order has one or many line items stored in the order_items table.
- products: the product’s information such as name, brand, category, model year, and list price. Each product belongs to a brand specified by the brand_id column. Hence, a brand may have zero or many products. Each product also belongs a category specified by the category_id column. Also, each category may have zero or many products.
- staffs: the essential information of staffs including first name, last name, email, and phone. A staff works at a store specified by the value in the store_id column. A store can have one or more staffs. A staff reports to a store manager specified by the value in the manager_id column. If the value in the manager_id is null, then the staff is the top manager.
- stocks: the inventory information i.e. the quantity of a particular product in a specific store.
- stores: the store’s information. Each store has a store name, contact information such as phone and email, and an address including street, city, state, and zip code.

If a staff no longer works for any stores, the value in the active column is set to zero.

## Setup
### Schema Diagram
The schema diagram used in this project can be viewed at the following link [here](https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.png). In this project, the dataset is splitted into two schemes, sales and productions.

### Create Database
```sql
CREATE DATABASE BikeStores;
USE BikeStores;
```
### Create and Load Database
In this project, SQL queries are executed using Microsoft SQL Server Management Studio Version 21. The database tables are created and loaded by running the SQL files provided from [www.sqlservertutorial.net](https://www.sqlservertutorial.net/wp-content/uploads/SQL-Server-Sample-Database.zip).

## SQL Queries
### Sales Performance Analysis
#### What was the total annual revenue?
```sql
ALTER TABLE sales.order_items
ADD total_price FLOAT;

UPDATE sales.order_items
SET total_price = (quantity*list_price - (1-list_price*discount));

SELECT 
YEAR(o.order_date) AS order_year,
SUM(i.total_price) AS total_revenue
FROM sales.orders AS o
JOIN sales.order_items AS i
ON o.order_id = i.order_id
GROUP BY
YEAR(o.order_date)
ORDER BY
total_revenue DESC;
```
Insight: The highest total revenue was in 2017 at $4,113,799.37, while the lowest was in 2018 at $2,162,584.14.

#### What was the total monthly revenue?
```sql
SELECT 
YEAR(o.order_date) AS order_year,
MONTH(o.order_date) AS order_month,
SUM(i.total_price) AS total_revenue
FROM sales.orders AS o
JOIN sales.order_items AS i
ON o.order_id = i.order_id 
GROUP BY YEAR(o.order_date), MONTH(o.order_date)
ORDER BY total_revenue DESC;
```

Insight: The highest total revenue was in April 2018 at $968,983.10, while the lowest was in June 2018 at $229.99.

#### What was the total revenue per category?
```sql
SELECT 
c.category_name,
SUM(i.total_price) AS total_revenue
FROM production.products AS p
JOIN production.categories AS c
ON p.category_id = c.category_id
JOIN sales.order_items AS i
ON p.product_id = i.product_id
GROUP BY c.category_name
ORDER BY SUM(i.total_price) DESC;
```
Insight: The highest total revenue by product was Mountain Bikes at $3,242,524.94, while the lowest was Children Bicycles at $350,537.07.

#### What was the total annual revenue for each category?
```sql
SELECT  
c.category_name,  
YEAR(o.order_date) AS order_year,  
SUM(i.total_price) AS total_revenue  
FROM sales.orders AS o  
JOIN sales.order_items AS i  
ON o.order_id = i.order_id  
JOIN production.products AS p  
ON i.product_id = p.product_id  
JOIN production.categories AS c  
ON p.category_id = c.category_id  
GROUP BY c.category_name, YEAR(o.order_date)  
ORDER BY  
total_revenue DESC;
```
Insight: The highest total annual revenue for a category was Mountain Bikes in 2016 at $1,418,225.17, while the lowest was Children Bicyles in 2018 at $68,780.53.


#### What was the total monthly revenue for each category?
```sql
SELECT 
c.category_name,
YEAR(o.order_date) AS order_year,
MONTH(o.order_date) AS order_month,
SUM(i.total_price) AS total_revenue
FROM sales.orders AS o
JOIN sales.order_items AS i
ON o.order_id = i.order_id 
JOIN production.products AS p
ON i.product_id = p.product_id
JOIN production.categories AS c
ON p.category_id = c.category_id
GROUP BY c.category_name, YEAR(o.order_date), MONTH(o.order_date)
ORDER BY total_revenue DESC;
```

Insight: Category with the highest total monthly revenue was Road Bikes in April 2018 at $329,669.94, while the lowest was Children Bicycles in June 2018 at $229.99.

#### What were the total units sold per year?
```sql
SELECT 
YEAR(o.order_date) AS order_year,
SUM(i.quantity) AS total_unit
FROM sales.orders AS o
JOIN sales.order_items AS i
ON o.order_id = i.order_id
GROUP BY YEAR(o.order_date)
ORDER BY total_unit DESC;
```

Insight: The highest number of units sold was in 2017 with 3,099 units, while the lowest was in 2018 with 1,316 units.


### Stores Performance Analysis
#### What were the total annual units sold per store?
```sql
SELECT  
o.store_id,  
YEAR(o.order_date) AS order_year,  
SUM(i.quantity) AS total_unit  
FROM sales.orders AS o  
JOIN sales.order_items AS i  
ON o.order_id = i.order_id  
GROUP BY o.store_id, YEAR(o.order_date)  
ORDER BY total_unit DESC;
```

Insight: The highest number of units sold by a store in a single year was 2,159 units by store_id = 2 in 2017, while the lowest was 149 units by store_id = 3 in 2018.

#### What was the total revenue per store?
```sql
SELECT  
s.store_name,
SUM(i.total_price) AS total_revenue
FROM sales.orders AS o
JOIN sales.stores AS s
ON o.store_id = s.store_id
JOIN sales.order_items AS i
ON o.order_id = i.order_id
GROUP BY s.store_name
ORDER BY SUM(i.total_price) DESC;
```

Insight: The highest revenue by store was Baldwin Bikes with $6,234,519.49, while the lowest was Rowlett Bikes with $1,025,731.74.

#### What was the total annual revenue per store?
```sql
SELECT  
s.store_name,
SUM(i.total_price) AS total_revenue,
YEAR(o.order_date) AS order_year
FROM sales.orders AS o
JOIN sales.stores AS s
ON o.store_id = s.store_id
JOIN sales.order_items AS i
ON o.order_id = i.order_id
GROUP BY s.store_name, YEAR(o.order_date)
ORDER BY total_revenue DESC;
```

Insight: The highest annual revenue by store was $2,957,523.49 by Baldwin Bikes in 2017, while the lowest was $226,682.98 by Rowlett Bikes in 2018.

#### What was the total stock per store?
```sql
SELECT
s.store_name,
SUM(sto.quantity) AS total_stock
FROM sales.stores AS s
JOIN production.stocks AS sto
ON s.store_id = sto.store_id
GROUP BY s.store_name
ORDER BY SUM(sto.quantity) DESC;
```

Insight: Rowlett Bikes had the highest total stock with 4,620 units, while Baldwin Bikes had the lowest with 4,359 units.

#### What was the most profitable state by total revenue?
```sql
SELECT  
str.state,  
SUM(i.quantity * (i.list_price * (1 - i.discount))) AS total_revenue  
FROM sales.stores AS str  
JOIN sales.orders AS o  
ON o.store_id = str.store_id  
JOIN sales.order_items AS i 
ON i.order_id = o.order_id  
GROUP BY str.state  
ORDER BY total_revenue DESC;
```

Insight: New York was the most profitable state with the highest total revenue with $5,215,751.28.

#### Which regions show the highest customer concentration for potential expansion or advertising?
```sql
SELECT 
c.state, 
COUNT(c.customer_id) AS number_of_people 
FROM sales.customers AS c 
GROUP BY c.state 
ORDER BY number_of_people DESC;
```

Insight: New York is the region with the highest customer concentration with 1,019 people, making it the ideal location for opening new stores.

### Products Performance Analysis
#### What were the total units sold per category?
```sql
SELECT 
c.category_name,
SUM(i.quantity) AS total_unit
FROM production.products AS p
JOIN production.categories AS c
ON p.category_id = c.category_id
JOIN sales.order_items AS i
ON p.product_id = i.product_id
GROUP BY c.category_name
ORDER BY SUM(i.quantity) DESC;
```

Insight: Cruisers Bicycles had the highest total units sold with 2,063 units, while Electric Bikes had the lowest with 315 units.

#### What were the total units sold per brand?
```sql
SELECT
b.brand_name,
SUM(i.quantity) AS total_orders
FROM production.products AS p 
JOIN production.brands AS b
ON p.brand_id = b.brand_id
JOIN sales.order_items AS i
ON p.product_id = i.product_id 
GROUP BY b.brand_name
ORDER BY SUM(i.quantity) DESC;
```

Insight: Electra recorded the highest total units sold with 2,612 units, while Strider had the lowest with 25 units.

#### What was the total stock per product?
```sql
SELECT 
p.product_name,
SUM(sto.quantity) AS total_stocks
FROM production.products AS p
JOIN production.stocks AS sto
ON p.product_id = sto.product_id
GROUP BY p.product_name
ORDER BY SUM(sto.quantity) DESC;
```

Insight: Electra Townie Original 7D - 2017 had the highest total stock with 125 units, while Trek Domane SLR Frameset - 2018 had the lowest with 5 units.

#### What was the total stock per category?
```sql
SELECT  
c.category_name,  
SUM(sto.quantity) AS total_stocks  
FROM production.stocks AS sto  
JOIN production.products AS p  
ON sto.product_id = p.product_id  
JOIN production.categories AS c  
ON p.category_id = c.category_id  
GROUP BY c.category_name  
ORDER BY total_stocks DESC;
```

Insight: Cruisers Bicycles had the highest total stock with 3,378 units, while Cyclocross Bicycles had the lowest with 414 units.

### Customers Behaviour Analysis
#### Who were the top 5 customers with the most spent?
```sql
ALTER TABLE sales.customers
ADD customer_name VARCHAR(30);

UPDATE sales.customers
SET customer_name = CONCAT(first_name, ' ', last_name);

SELECT TOP 5
cu.customer_name,
SUM(i.total_price) AS total_revenue
FROM sales.orders AS o
JOIN sales.customers AS cu  
ON o.customer_id = cu.customer_id
JOIN sales.order_items AS i  
ON o.order_id = i.order_id
GROUP BY cu.customer_name
ORDER BY SUM(i.total_price) DESC;
```

Insight: The top 5 customers based on their spent were Pamelia Newman, Abby Gamble, Sharyn Hopkins, Lyndsey Bean, and Emmitt Sanchez.

#### Who were the top 5 customers with the most total units ordered?
```sql
SELECT TOP 5
cu.customer_name,
SUM(i.quantity) AS total_orders
FROM sales.orders AS o
JOIN sales.customers AS cu 
ON o.customer_id = cu.customer_id
JOIN sales.order_items AS i
ON o.order_id = i.order_id
GROUP BY cu.customer_name
ORDER BY SUM(i.quantity) DESC;
```

Insight: the top 5 customers based on their total units ordered were Emmitt Sanchez, Tameka Fisher, Pamelia Newman, Debra Burks, and Elinore Aguilar.

#### Did customer retention perform well by store and in total?
```sql
SELECT  
store_name,  
COUNT(CASE WHEN status = 'returning' THEN 1 END) AS returning_customers,  
COUNT(CASE WHEN status = 'new' THEN 1 END) AS new_customers,  
CAST(ROUND(100.0 * COUNT(CASE WHEN status = 'returning' THEN 1 END) / COUNT(*), 2) 
AS VARCHAR) + '%' AS returning_percentage  
FROM (
SELECT  
cu.customer_id,  
s.store_name,  
CASE  
WHEN COUNT(DISTINCT o.order_id) > 1 THEN 'returning'  
ELSE 'new'  
END AS status  
FROM sales.customers AS cu  
JOIN sales.orders AS o  
ON o.customer_id = cu.customer_id  
JOIN sales.stores AS s  
ON s.store_id = o.store_id  
GROUP BY cu.customer_id, s.store_name) 
AS customer_status_by_store  
GROUP BY store_name  
ORDER BY CAST(ROUND(100.0 * COUNT(CASE WHEN status = 'returning' THEN 1 END) / COUNT(*), 2) 
AS VARCHAR) DESC;
```

Insight: Santa Cruz Bikes had the largest retention rate of customers with 18.31%.

### Staff Performance Analysis
#### Which staff generated the highest total revenue?
```sql
WITH staff_name_data AS(
SELECT
staff_id,
first_name,
last_name,
CONCAT(first_name, ' ', last_name) AS staff_name
FROM 
sales.staffs
)
SELECT TOP 5
st.staff_name,
s.store_name,
SUM(i.total_price) AS total_revenue
FROM sales.orders AS o 
JOIN staff_name_data AS st
ON o.staff_id = st.staff_id
JOIN sales.stores AS s
ON o.store_id = s.store_id
JOIN sales.order_items AS i
ON o.order_id = i.order_id
GROUP BY st.staff_name, s.store_name
ORDER BY SUM(i.total_price) DESC;
```

Insight: Marcelene Boyer from Baldwin Bikes generated the highest revenue, totaling $3,147,789.21.

### Order Overview
#### What were all the order status?
```sql
SELECT  
order_status,  
COUNT(*) AS total_orders,  
COUNT(*) * 100.0 / (SELECT COUNT(*) FROM sales.orders) AS percentage_order_status  
FROM sales.orders  
GROUP BY order_status  
ORDER BY total_orders DESC;
```

Insight: All order were 89.47% completed.

###Recommendations
- 
