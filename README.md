# Pizza Sales SQL
## 1.0 Project Overview

**1.1 Business Goals** : 
The core analytical goal of this project is to derive meaningful business insights from the sales data. 
This involves identifying key performance indicators (KPIs) such as total revenue and order volume,
understanding customer purchasing patterns related to time and product choice, 
and determining the most popular and profitable products to inform strategic decisions regarding inventory, marketing, and menu optimization.

**1.2 Tools & Technologies** :
The analysis was performed using the following tools:
* MySQL Workbench: for database management, schema creation, and executing SQL queries.

--------------------------------------------------------------------------------
## 2.0 Dataset Description

A clear understanding of the data's structure is fundamental to any successful analysis. This project utilizes a dataset composed of four distinct CSV files, which were imported and structured into relational tables within a MySQL database. This relational model is crucial for performing complex joins and aggregations.

The database schema consists of the following four tables:
| Table Name        | Description                                                                   | Key Columns                                            |
| ----------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------ |
| **pizzas**        | Details for each unique pizza SKU, linking `pizza_type_id` to size and price. | `pizza_id`, `pizza_type_id`, `size`, `price`           |
| **pizza_types**   | Lookup table containing pizza type details such as category and ingredients.  | `pizza_type_id`, `name`, `category`, `ingredients`     |
| **orders**        | Records of each customer order, with date and time stamps.                    | `order_id`, `order_date`, `order_time`                 |
| **order_details** | Junction table linking orders to pizzas, including quantities ordered.        | `order_details_id`, `order_id`, `pizza_id`, `quantity` |

--------------------------------------------------------------------------------
## 3.0 Methodology: From Raw Data to Actionable Insights
The project followed a systematic methodology to ensure a thorough and accurate analysis. The process began with setting up the database environment, followed by importing the raw data, and culminated in the execution of a series of targeted SQL queries designed to address specific business questions.
1. **Database and Table Creation:** A new database schema named pizzahut was created in MySQL Workbench. Subsequently, the four tables—orders, order_details, pizzas, and pizza_types—were defined using CREATE TABLE statements. This step established the relational structure, data types, and constraints (e.g., PRIMARY KEY, NOT NULL) necessary for data integrity.
2. **Data Import:** The MySQL Table Data Import Wizard was used to load the data from the four source CSV files into their corresponding tables. This process successfully imported a total of 21,350 records into the orders table and 48,620 records into the order_details table, populating the database for analysis.
3. **Structured Querying:** A comprehensive set of SQL queries was developed to perform the analysis. These queries were intentionally structured to progress from basic key performance indicators to more advanced, complex insights, demonstrating a layered analytical approach to answering key business questions.
This structured methodology provides a clear path from raw data to the final findings presented in the detailed analysis section.
--------------------------------------------------------------------------------
## 4.0 Data Analysis & SQL Queries
This core section presents the specific business questions addressed during the analysis and the corresponding SQL queries used to derive the answers. The analysis is organized into three parts—Basic, Intermediate, and Advanced—to showcase a progression from foundational metrics to deeper, more nuanced business insights.

### 4.1 Basic KPIs

**1. Total Orders Placed**
```sql
SELECT
  COUNT(order_id) AS total_orders
FROM orders;
```
Insight: A total of 21,350 orders were placed, establishing the overall transaction volume.


**2. Total Revenue Generated**
```sql
SELECT
  ROUND(SUM(order_details.quantity * pizzas.price), 2) AS total_revenue
FROM order_details
JOIN pizzas
  ON pizzas.pizza_id = order_details.pizza_id;
```
Insight: The total revenue generated from pizza sales is $817,860.05, providing a key metric for overall business performance.


**3. Highest-Priced Pizza**
```sql
SELECT
  pizza_types.name,
  pizzas.price
FROM pizza_types
JOIN pizzas
  ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;
```
Insight: The highest-priced pizza is "The Greek Pizza", which can be highlighted in premium marketing initiatives.


**4. Most Common Pizza Size Ordered**
```sql
SELECT
  pizzas.size,
  COUNT(order_details.order_details_id) AS order_count
FROM pizzas
JOIN order_details
  ON pizzas.pizza_id = order_details.pizza_id
GROUP BY
  pizzas.size
ORDER BY
  order_count DESC;
```
Insight: The "Large" size is the most frequently ordered, suggesting that inventory and marketing efforts should prioritize this size.


**5. Top 5 Most-Ordered Pizza Types**
```sql
SELECT
  pizza_types.name,
  SUM(order_details.quantity) AS quantity
FROM pizza_types
JOIN pizzas
  ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
  ON order_details.pizza_id = pizzas.pizza_id
GROUP BY
  pizza_types.name
ORDER BY
  quantity DESC
LIMIT 5;
```
Insight: The top 5 most-ordered pizzas are "The Classic Deluxe," "The Barbecue Chicken," "The Hawaiian," "The Pepperoni," and "The Thai Chicken," indicating these are core products driving sales volume.


### 4.2 Intermediate Analysis

**6. Total Quantity of Each Pizza Category Ordered**
```sql
SELECT
  pizza_types.category,
  SUM(order_details.quantity) AS quantity
FROM pizza_types
JOIN pizzas
  ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
  ON order_details.pizza_id = pizzas.pizza_id
GROUP BY
  pizza_types.category
ORDER BY
  quantity DESC;
```  
Insight: The 'Classic' category is the most popular, with 14,888 pizzas ordered. It significantly outperforms the 'Supreme' (11,987), 'Veggie' (11,648), and 'Chicken' (11,050) categories, indicating a strong customer preference for traditional options.


**7. Distribution of Orders by Hour of the Day**
```sql
SELECT
  HOUR(order_time) AS hour,
  COUNT(order_id) AS order_count
FROM orders
GROUP BY
  HOUR(order_time)
ORDER BY
  hour;
```
Insight: The busiest hours for orders are concentrated during the lunch peak (12 PM - 1 PM) and the early evening, which can inform staffing schedules and promotional timing.


**8. Category-wise Distribution of Pizzas**
```sql
SELECT
  category,
  COUNT(name)
FROM pizza_types
GROUP BY
  category;
```
Insight: The menu offers a diverse selection, with 9 'Supreme' and 9 'Veggie' pizzas, 8 'Classic' options, and 6 'Chicken' varieties. This balanced distribution caters to a wide range of customer preferences.


**9. Average Number of Pizzas Ordered Per Day**
```sql
SELECT
  ROUND(AVG(quantity), 0)
FROM (SELECT
  orders.order_date,
  SUM(order_details.quantity) AS quantity
FROM orders
JOIN order_details
  ON orders.order_id = order_details.order_id
GROUP BY
  orders.order_date) AS order_quantity;
``` 
Insight: On average, 138 pizzas are ordered per day, providing a baseline for daily sales forecasting and inventory management.


**10. Top 3 Most-Ordered Pizza Types Based on Revenue**
```sql
SELECT
  pizza_types.name,
  SUM(order_details.quantity * pizzas.price) AS revenue
FROM pizza_types
JOIN pizzas
  ON pizzas.pizza_type_id = pizza_types.pizza_type_id
JOIN order_details
  ON order_details.pizza_id = pizzas.pizza_id
GROUP BY
  pizza_types.name
ORDER BY
  revenue DESC
LIMIT 3;
```
Insight: The top three revenue-generating pizzas are "The Thai Chicken Pizza," "The Barbecue Chicken Pizza," and "The California Chicken Pizza," highlighting the profitability of specialty chicken pizzas.


### 4.3 Advanced Insights

**11. Percentage Contribution of Each Pizza Category to Total Revenue**
```sql
SELECT
  pizza_types.category,
  ROUND(SUM(order_details.quantity * pizzas.price) / (SELECT
    ROUND(SUM(order_details.quantity * pizzas.price), 2)
  FROM order_details
  JOIN pizzas
    ON pizzas.pizza_id = order_details.pizza_id) * 100, 2) AS revenue
FROM pizza_types
JOIN pizzas
  ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
  ON order_details.pizza_id = pizzas.pizza_id
GROUP BY
  pizza_types.category
ORDER BY
  revenue DESC;
```
Insight: The 'Classic' category contributes the most to total revenue at 26.91%, followed closely by 'Supreme' (25.42%), 'Chicken' (23.96%), and 'Veggie' (23.69%), showing a relatively balanced revenue distribution across categories.


**12. Cumulative Revenue Over Time**
```sql
SELECT
  order_date,
  SUM(revenue) OVER (ORDER BY order_date) AS cum_revenue
FROM (SELECT
  orders.order_date,
  SUM(order_details.quantity * pizzas.price) AS revenue
FROM order_details
JOIN pizzas
  ON order_details.pizza_id = pizzas.pizza_id
JOIN orders
  ON orders.order_id = order_details.order_id
GROUP BY
  orders.order_date) AS sales;
```  
Insight: The cumulative revenue analysis shows a steady growth trajectory, which is crucial for forecasting and assessing performance against quarterly financial targets. It confirms the business is maintaining positive momentum.


**13. Top 3 Most-Ordered Pizzas by Revenue for Each Category**
```sql
SELECT
  name,
  revenue
FROM (SELECT
  category,
  name,
  revenue,
  RANK() OVER (PARTITION BY category ORDER BY revenue DESC) AS rn
FROM (SELECT
  pizza_types.category,
  pizza_types.name,
  SUM((order_details.quantity) * pizzas.price) AS revenue
FROM pizza_types
JOIN pizzas
  ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
  ON order_details.pizza_id = pizzas.pizza_id
GROUP BY
  pizza_types.category,
  pizza_types.name) AS a) AS b
WHERE rn <= 3;
```
Insight: This query identifies the top revenue performers within each category, providing granular detail for menu optimization. Key findings include:
- Chicken: The Thai Chicken Pizza, The Barbecue Chicken Pizza, The California Chicken Pizza.
- Classic: The Classic Deluxe Pizza, The Hawaiian Pizza, The Pepperoni Pizza.
- Supreme: The Spicy Italian Pizza, The Italian Supreme Pizza, The Sicilian Pizza.
- Veggie: The Four Cheese Pizza, The Mexicana Pizza, The Five Cheese Pizza.


--------------------------------------------------------------------------------

## 5.0 Summary of Findings & Conclusion

This final section consolidates the key business insights uncovered during the SQL analysis, offering a strategic summary of the findings and their potential impact. The analysis successfully translated raw sales data into a clear narrative of business performance.

### Main Findings

- **Peak Hours:** The highest volume of orders occurs during the midday lunch rush, specifically between **12:00 PM and 1:00 PM**.
- **Sales Drivers:** The **'Large'** pizza size is the most popular choice among customers. By category, the **'Classic'** selection drives the highest order volume, indicating strong customer preference for traditional options.
- **Revenue Leaders:** Despite the **'Classic'** category's popularity in volume, the top three highest revenue-generating pizzas are **"The Thai Chicken Pizza," "The Barbecue Chicken Pizza,"** and **"The California Chicken Pizza,"** highlighting the profitability of premium or specialty items.
- **Category Performance:** The **'Classic'** category is the largest contributor to total sales, accounting for **26.91%** of all revenue, closely followed by the **'Supreme'** category.

### Conclusion

This project demonstrates the power of **SQL** in extracting strategic value from transactional data. The insights generated can directly inform critical business decisions, including:

- Optimizing staffing during peak hours  
- Managing inventory for popular items  
- Developing marketing campaigns that highlight high-revenue products  

This analytical foundation provides the pizza business with a **data-driven path toward enhanced efficiency and profitability**.
