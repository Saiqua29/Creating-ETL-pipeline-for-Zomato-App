# 🍽️ Creating ETL Pipeline for Zomato App: Joining the Pieces to Tell a Story

Zomato is a popular food delivery app in India. Starting as **Foodiebay** in 2008, it was rebranded to **Zomato** in 2010 by Deepinder Goyal and Pankaj Chaddha. Over the past 16 years, Zomato has revolutionized the food delivery ecosystem in India, now boasting a market cap of ₹2.529 Trillion (as of September 2024).

## 📌 Project Overview

This project demonstrates how to create an **ETL (Extract, Transform, Load)** pipeline using Zomato restaurant data. The aim is to transform raw inputs into structured databases, which are then queried to derive meaningful insights. These insights can help stakeholders in making informed business decisions such as:

- Expansion opportunities
- City and locality analysis
- Performance and segmentation studies
- Cuisine trends

### 🧠 Key Outcomes

- Learn how to clean and preprocess real-world data using Python.
- Store structured data into **MySQL**.
- Build complex **SQL queries** to answer real business questions.
- Present findings via **Power BI dashboard**.

---

## 🔁 Introduction to ETL

ETL stands for **Extract, Transform, Load**.

1. **Extract**: Read raw `.xlsx` data using Python.
2. **Transform**: Clean and engineer features relevant for analysis.
3. **Load**: Upload structured tables to a MySQL database.

---

## 📂 Dataset Information

- **Source**: [Zomato Restaurants Dataset on Kaggle](https://www.kaggle.com/datasets/shrutimehta/zomato-restaurants-data)
- Originally scraped from Zomato’s Developer API(now obsolete)

- **Format**: Excel (.xlsx)
- **Scope**: Global restaurant data, but filtered to **India only** for this project.

### ⚙️ Key Fields

- `restaurant_id`, `restaurant_name`, `city`, `locality`, `latitude`, `longitude`
- `cuisines`, `average_cost_for_two`, `has_table_booking`, `has_online_delivery`
- `price_range`, `aggregate_rating`, `rating_text`, `votes`, etc.

---

## 🔧 Data Preprocessing & Feature Engineering

- Filtered data for **India** (`country_code == 1`)
- Removed redundant columns like `address`, `currency`, etc.
- Removed columns with minimal relevance: `switch_to_order_menu`, `is_delivering_now`
- Handled multi-cuisine data by splitting `cuisines` column into 8 separate columns
- Normalized column names (lowercase, underscores, no spaces)
- Split dataset into two tables:
- `restaurant_info`: Basic details
- `cuisine_info`: Cuisine-specific information

---

## 🛢️ Python to MySQL Database Integration

- Database Name: `zomatoapp`
- Tables:
- `restaurant_info` (Primary Key: `restaurant_id`)
- `cuisine_info` (Primary Key: `restaurant_id`)
- Used `mysql.connector` and `sqlalchemy` to connect and load data

(A separate read Me file exists for EDA in python)


---

 ## 🧮 SQL Queries & Insights

 ## 📊 City Analysis
 ```sql
SELECT 
    ri.city,
    ri.restaurant_name,
    ri.votes
FROM 
    restaurant_info ri
JOIN (
    SELECT 
        city,
        MAX(votes) AS max_votes
    FROM 
        restaurant_info
    GROUP BY 
        city
) AS max_votes_per_city
ON 
    ri.city = max_votes_per_city.city
    AND ri.votes = max_votes_per_city.max_votes
ORDER BY 
    ri.city;
```
## 🍴 Cuisine Analysis

    Created a stored procedure UnpivotCuisines() to normalize multiple cuisine columns using UNION ALL
    (A separate read Me file exists for code & logic of the stored procedure UnpivotCuisines())

    Identified:

    - Top 3 cuisines per city

    - Specialty restaurants (only one cuisine)

    - Most common cuisine in India (e.g., North Indian)

1. **Finding top 3 cuisines from each city**
 ```sql
 CALL UnpivotCuisines();
WITH CuisinePopularity AS (
    SELECT 
        city,
        cuisine,
        COUNT(*) AS cuisine_count
    FROM 
        UnpivotedCuisines
    GROUP BY 
        city, 
        cuisine
),

RankedCuisines AS (
    SELECT 
        city,
        cuisine,
        cuisine_count,
        DENSE_RANK() OVER (PARTITION BY city ORDER BY cuisine_count DESC) AS ranking
    FROM 
        CuisinePopularity
)

SELECT 
    city,
    cuisine,
    cuisine_count
FROM 
    RankedCuisines
WHERE 
    ranking <= 3
ORDER BY 
    city, ranking;
```

2. **Querying restaurants having only 1 unique cuisine i.e specialty restaurants**
 ```sql
 SELECT 
    ri.restaurant_name,
    ri.city,
    MIN(uc.cuisine) AS cuisine
FROM 
    restaurant_info ri 
JOIN 
    UnpivotedCuisines uc ON ri.restaurant_id = uc.restaurant_id
GROUP BY 
    ri.restaurant_name, 
    ri.city
HAVING 
    COUNT(DISTINCT uc.cuisine) = 1
ORDER BY 
    ri.city, 
    ri.restaurant_name;
```
3. **Query to Find the Most Common Cuisine in India:**
This query shows the most common cuisine in India is 'North Indian'   

```sql
CALL UnpivotCuisines();

SELECT 
    cuisine,
    COUNT(*) AS cuisine_count
FROM 
    UnpivotedCuisines
GROUP BY 
    cuisine
ORDER BY 
    cuisine_count DESC
LIMIT 1;
```
## Expansion Opportunities ##
1. **Finding cities with Few Restaurants but high ratings**
```sql
SELECT city, COUNT(*) as restaurant_count, AVG(aggregate_rating) as avg_rating
FROM restaurant_info
GROUP BY city
HAVING restaurant_count < 10 AND avg_rating > 4.0
ORDER BY avg_rating DESC;
```

## Operational Insights ##
1. **Relationship Between Price Range and Availability of Table Booking or Online Delivery**
Shows the total number of restaurants within a given price range with 1 being lowest & 4 being highest. As gives corresponding table booking & online delivery service counts of restaurants.


```sql
SELECT
    price_range,
    SUM(CASE WHEN has_table_booking = 'Yes' THEN 1 ELSE 0 END) AS table_booking_count,
    SUM(CASE WHEN has_online_delivery = 'Yes' THEN 1 ELSE 0 END) AS online_delivery_count,
    COUNT(*) AS total_restaurants
FROM
    restaurant_info
GROUP BY
    price_range
ORDER BY
    price_range;
```


## Competitive Analysis ##
1. **How do the ratings and pricing of restaurants compare across different cuisines?**

```sql
CALL UnpivotCuisines();

-- Step 2: Compare the ratings and pricing of restaurants across different cuisines
SELECT 
    uc.cuisine,
    AVG(ri.aggregate_rating) AS avg_rating,
    AVG(ri.price_range) AS avg_price_range,
    COUNT(DISTINCT ri.restaurant_id) AS num_restaurants
FROM 
    UnpivotedCuisines uc
JOIN 
    restaurant_info ri ON uc.restaurant_id = ri.restaurant_id
GROUP BY 
    uc.cuisine
ORDER BY 
    avg_rating DESC, avg_price_range DESC;
```


## 🖼️ Visualizations

![Zomato India KPI Dashboard Screenshot](https://github.com/Saiqua29/Creating-ETL-pipeline-for-Zomato-App/blob/main/dashboard%20screenshot.png)

---
📸
*Above: Power BI dashboard screenshot visualizing key metrics and city-level insights.*

---

## 🌍 Future Scope

- Incorporate real-time API-based location mapping.
- Perform clustering to group users based on food preferences and location.
- Integrate machine learning to predict restaurant performance.

---

## 📦 Zomato_ETL_Pipeline
├── Dataset from kaggle              
├── EDA in python             
├── Creating database,tables & inserting table data from python into SQL              
├── Connecting SQL queries to allow for query folding,creating PowerBI dashboard   
             

## 🧰 Tech Stack

   - Python (Pandas, SQLAlchemy, MySQL Connector)

   - MySQL

   - Power BI

   - Kaggle Dataset

## 🎓 Educational Use

This project is intended for educational and analytical purposes only. Data is sourced from a publicly available Kaggle dataset that reflects real-world scenarios.

## 📬 Contact

If you have any questions, feel free to connect via GitHub Issues or email.
- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/saiqua-shaikh-b28682124/)

## ⭐ Star the Repository

If you found this project helpful, consider giving it a ⭐ to support more such initiatives!
