# 📇 Retail-Packaging-Impact-Analysis-SQL
🔦 This project explores the dataset of an **online supermarket** that specialises in fresh produce, analyzes **sales performance**, and investigates the impact of sustainable packaging change in all the company's areas. Utilizes **Google BigQuery and SQL.**

<img width="800" height="800" alt="Cover photo" src="https://github.com/user-attachments/assets/1c0c8ec7-b337-420c-a0c9-fdb681dbaa2e" />

- Author: Huy Huynh
- Date: October 2025
- Tools: BigQuery, SQL

## 🚀 Project Overview
- Data Mart is Danny’s latest online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his **sales performance**.
- In **June 2020**, large-scale supply changes were made. All Data Mart products now use **sustainable packaging** methods.
- Danny needs your help to **quantify the impact** of this change on the **sales performance** for Data Mart and it’s separate business areas.
- The key business question he wants you to help him answer are the following:
  - What was the quantifiable impact of the changes introduced in June 2020?
  - Which platform, region, segment and customer types were the most impacted by this change?
  - What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

## 📠 Data Sources
- The dataset is the case study #5 of Dany's 8-week SQL challenges, created in June 2021.
-  Check out the Case Study to get access to the datase:
    [Case Study]('https://8weeksqlchallenge.com/case-study-5/')
- For this case study there is only a single table: data_mart.weekly_sales

<img width="336" height="400" alt="case-study-5-erd" src="https://github.com/user-attachments/assets/09fb3701-4263-4c78-8f45-52ab6d6a0273" />

- 🗃️ **Example Rows:**

| week_date  | region  | platform  | segment  | customer_type  | transactions  | sales       |
|------------|---------|-----------|----------|----------------|---------------|-------------|
| 9/9/20     | OCEANIA | Shopify   | C3       | New            | 610           | 110033.89   |
| 29/7/20    | AFRICA  | Retail    | C1       | New            | 110692        | 3053771.19  |
| 22/7/20    | EUROPE  | Shopify   | C4       | Existing       | 24            | 8101.54     |
| 13/5/20    | AFRICA  | Shopify   | null     | Guest          | 5287          | 1003301.37  |
| 24/7/19    | ASIA    | Retail    | C1       | New            | 127342        | 3151780.41  |

- 📒 **Column Dictionary:**
1. Data Mart has **international** operations using a multi-region strategy
2. Data Mart has both, a **retail and online** platform in the form of a Shopify storefront to serve their customers
3. Customer **segment** and **customer_type** data relate to personal **age and demographics** information that is shared with Data Mart
4. **Transactions** is the count of **unique purchases** made through Data Mart and **sales** is the actual **dollar amount** of purchases

## ⚒️ Tools:
- SQL - Google BigQuery.

## 📖 Main Process:
### 🛁 1. Data cleansing steps
**📆 a. Create Calendar Table**
- Convert the *week_date* to a **DATE** format
- Add a _week_number_ as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a _month_number_ with the calendar month for each week_date value as the 3rd column
- Add a _calendar_year_ column as the 4th column containing either 2018, 2019 or 2020 values

```SQL
WITH calendar AS(
    SELECT  DISTINCT week_date, 
            EXTRACT(WEEK FROM week_date) AS week_num,
            EXTRACT (MONTH FROM week_date) AS month,
            EXTRACT (YEAR FROM week_date) AS year,
    FROM `bigquerrysandbox-416110.data_mart.weekly_sales` 
    ORDER BY week_date DESC
),
```
- **calendar table** random 5 rows:

| Row | week_date   | week_num | month  | year |
|-----|-------------|----------|--------|------|
| 1   | 2020-08-31  | 35       | 8      | 2020 |
| 2   | 2020-08-24  | 34       | 8      | 2020 |
| 3   | 2020-08-17  | 33       | 8      | 2020 |
| 4   | 2020-08-10  | 32       | 8      | 2020 |
| 5   | 2020-08-03  | 31       | 8      | 2020 |

**🏷️ b. Create Segment Detail Table**
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

segment  |	age_band
---------|-----------
1	       | Young Adults
2	       | Middle Aged
3 or 4	 | Retirees

- Add a new demographic column using the following mapping for the first letter in the segment values:

segment	 | demographic
---------|------------
C	       | Couples
F        | Families

```SQL
     segment_detail AS(
    SELECT  DISTINCT 
            CASE WHEN segment = 'null' THEN 'unknow'
                 ELSE segment 
            END AS segment,
            CASE WHEN segment LIKE '%1' THEN 'Young Adults'
                 WHEN segment LIKE '%2' THEN 'Middle Ages'
                 WHEN segment LIKE '%3' OR segment LIKE '%4'
                    THEN 'Retirees'
                 ELSE 'unknow' 
            END AS age_band,
            CASE WHEN segment LIKE 'C%' THEN 'Couple'
                 WHEN segment LIKE 'F%' THEN 'Family'
                 ELSE 'unknow'  
            END AS demographic
    FROM `bigquerrysandbox-416110.data_mart.weekly_sales`
),
```
- **segment_detail** table random 5 rows:

| Row | segment  | age_band      | demographic |
|-----|----------|---------------|--------------|
| 1   | C1       | Young Adults  | Couple       |
| 2   | C2       | Middle Ages   | Couple       |
| 3   | C3       | Retirees      | Couple       |
| 4   | C4       | Retirees      | Couple       |
| 5   | F1       | Young Adults  | Family       |

**🖋️ c. Insert column into weekly_sales and turn it into"weekly_sales_use""**
- Ensure all _null_ string values with an _"unknown"_ string value in the original segment column as well as the new age_band and demographic columns
- Generate a new _avg_transaction_ column as the sales value divided by transactions, rounded to 2 decimal places for each record\
  
```SQL
    weekly_sales_use AS(
    SELECT  * EXCEPT(segment),
            CASE WHEN segment = 'null' THEN 'unknow'
                 ELSE segment 
            END AS segment,
            ROUND(sales/transactions,2) AS avg_tranaction_value
    FROM `bigquerrysandbox-416110.data_mart.weekly_sales`
)
```
### 👓 2. Data Exploration
**1. What day of the week is used for each week_date value?**
```SQL
SELECT  DISTINCT 
            FORMAT_DATE('%A', week_date) AS day_name 
FROM calendar;
```
- Output:

Row	| day_name
----|---------
1	  | Monday

- Only **Monday** is used for each week_date value

**2. What range of week numbers are missing from the dataset?**
```SQL
    # The idea is to check the week_num that does not appear in the dataset
     exist_week AS(
    SELECT DISTINCT week_num
    FROM calendar
),
    # Assume that there is 53 weeks in a year
     all_week AS( 
    SELECT week_num
    FROM UNNEST(GENERATE_ARRAY(1,53)) AS week_num
),
     missing_week AS(
    SELECT a.week_num AS miss_week, 
            e.week_num   
    FROM all_week AS a    
    LEFT JOIN exist_week AS e 
    USING (week_num)
    WHERE e.week_num IS NULL
)
SELECT STRING_AGG(CAST(miss_week AS STRING), ', ') AS missed 
FROM missing_week
```
- Output:

Row	 | missed
-----|---------------
1    | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53

-> There are 2 range of week that missing sale value, from week **1 -> 11** and week **36 -> 53**

**3. How many total transactions were there for each year in the dataset?**
```SQL
SELECT  c.year,
        FORMAT("%'d", SUM(w.transactions)) AS total_transactions
FROM weekly_sales_use w
LEFT JOIN calendar c
USING (week_date)
GROUP BY c.year
```
- Output:

| Row | year | total_transactions |
|-----|------|--------------------|
| 1   | 2020 | 375,813,651        |
| 2   | 2019 | 365,639,285        |
| 3   | 2018 | 346,406,460        |

-> Number of transactions seems to **grow** **5.6%** in 2019 and **5.5%** in 2020 

**4. What is the total sales for each region for each month?**
```SQL
SELECT  w.region,
        c.month,
        FORMAT("%'d",SUM(sales)) AS total_sales
FROM weekly_sales_use w
LEFT JOIN calendar c 
USING(week_date)
GROUP BY w.region,
         c.month 
ORDER BY w.region,
         c.month
```
- Output: 
<details>
<summary>📊 Monthly Sales by Region (click to expand)</summary>
  
| Row | region        | month | total_sales   |
|-----|---------------|-------|---------------|
| 1   | AFRICA        | 3     | 567,767,480   |
| 2   | AFRICA        | 4     | 1,911,783,504 |
| 3   | AFRICA        | 5     | 1,647,244,738 |
| 4   | AFRICA        | 6     | 1,767,559,760 |
| 5   | AFRICA        | 7     | 1,960,219,710 |
| 6   | AFRICA        | 8     | 1,809,596,890 |
| 7   | AFRICA        | 9     | 276,320,987   |
| 8   | ASIA          | 3     | 529,770,793   |
| 9   | ASIA          | 4     | 1,804,628,707 |
| 10  | ASIA          | 5     | 1,526,285,399 |
| 11  | ASIA          | 6     | 1,619,482,889 |
| 12  | ASIA          | 7     | 1,768,844,756 |
| 13  | ASIA          | 8     | 1,663,320,609 |
| 14  | ASIA          | 9     | 252,836,807   |
| 15  | CANADA        | 3     | 144,634,329   |
| 16  | CANADA        | 4     | 484,552,594   |
| 17  | CANADA        | 5     | 412,378,365   |
| 18  | CANADA        | 6     | 443,846,698   |
| 19  | CANADA        | 7     | 477,134,947   |
| 20  | CANADA        | 8     | 447,073,019   |
| 21  | CANADA        | 9     | 69,067,959    |
| 22  | EUROPE        | 3     | 35,337,093    |
| 23  | EUROPE        | 4     | 127,334,255   |
| 24  | EUROPE        | 5     | 109,338,389   |
| 25  | EUROPE        | 6     | 122,813,826   |
| 26  | EUROPE        | 7     | 136,757,466   |
| 27  | EUROPE        | 8     | 122,102,995   |
| 28  | EUROPE        | 9     | 18,877,433    |
| 29  | OCEANIA       | 3     | 783,282,888   |
| 30  | OCEANIA       | 4     | 2,599,767,620 |
| 31  | OCEANIA       | 5     | 2,215,657,304 |
| 32  | OCEANIA       | 6     | 2,371,884,744 |
| 33  | OCEANIA       | 7     | 2,563,459,400 |
| 34  | OCEANIA       | 8     | 2,432,313,652 |
| 35  | OCEANIA       | 9     | 372,465,518   |
| 36  | SOUTHAMERICA  | 3     | 71,023,109    |
| 37  | SOUTHAMERICA  | 4     | 238,451,531   |
| 38  | SOUTHAMERICA  | 5     | 201,391,809   |
| 39  | SOUTHAMERICA  | 6     | 218,247,455   |
| 40  | SOUTHAMERICA  | 7     | 235,582,776   |
| 41  | SOUTHAMERICA  | 8     | 221,166,052   |
| 42  | SOUTHAMERICA  | 9     | 34,175,583    |
| 43  | USA           | 3     | 225,353,043   |
| 44  | USA           | 4     | 759,786,323   |
| 45  | USA           | 5     | 655,967,121   |
| 46  | USA           | 6     | 703,878,990   |
| 47  | USA           | 7     | 760,331,754   |
| 48  | USA           | 8     | 712,002,790   |
| 49  | USA           | 9     | 110,532,368   |

</details>

- **April and July** seem to have the highest total sales over time in every region.
- March and September have very few total sales, but it may be due to the lack of records for these 2 months.

-> Need to gather more information for the whole year to gain more precise information.

**5. What is the total count of transactions for each platform**
```SQL
SELECT  platform,
        FORMAT("%'d", SUM(transactions)) AS total_transactions
FROM weekly_sales_use 
GROUP BY platform
```
- Output:

Row  | platform	| total_transactions
-----|----------|--------
1	   | Shopify	| 5,925,169
2	   | Retail	  | 1,081,934,227

- **Retail** holds a huge share of the total number of transactions ( **~99.4%** ).

**6. What is the percentage of sales for Retail vs Shopify for each month?**
```SQL
     month_sales AS(
    SELECT  c.month,
            SUM(sales) AS total_sales_month
    FROM weekly_sales_use w
    LEFT JOIN calendar c
    USING (week_date)
    GROUP BY c.month
),
     plat_sales AS(
    SELECT  w.platform,
            c.month,
            SUM(sales) AS total_sales_plat
    FROM weekly_sales_use w
    LEFT JOIN calendar c
    USING(week_date)
    GROUP BY c.month, 
             w.platform
)
SELECT  p.month,
        p.platform,
        ROUND(p.total_sales_plat*100/m.total_sales_month,2) AS sales_pct
FROM plat_sales p
LEFT JOIN month_sales m
USING(month)
ORDER BY p.month, p.platform
```
- Output:

| Row | month  | platform  | sales_pct  |
|-----|--------|-----------|------------|
| 1   | 3      | Retail    | 97.54      |
| 2   | 3      | Shopify   | 2.46       |
| 3   | 4      | Retail    | 97.59      |
| 4   | 4      | Shopify   | 2.41       |
| 5   | 5      | Retail    | 97.3       |
| 6   | 5      | Shopify   | 2.7        |
| 7   | 6      | Retail    | 97.27      |
| 8   | 6      | Shopify   | 2.73       |
| 9   | 7      | Retail    | 97.29      |
| 10  | 7      | Shopify   | 2.71       |
| 11  | 8      | Retail    | 97.08      |
| 12  | 8      | Shopify   | 2.92       |
| 13  | 9      | Retail    | 97.38      |
| 14  | 9      | Shopify   | 2.62       |

- The **Retail** share of revenue is dominant throughout every month, above **97%** compared to **2%** of **Shopify**.

**7. What is the percentage of sales by demographic for each year in the dataset?**
```SQL
,    
     demogra_sales AS(       
    SELECT  s.demographic,
            c.year,
            SUM(w.sales) total_demogra_sales
    FROM weekly_sales_use w
    LEFT JOIN segment_detail s
    USING(segment) 
    LEFT JOIN calendar c
    USING(week_date)
    GROUP BY s.demographic,
             c.year
),
     year_sales AS(
    SELECT  c.year,
            SUM(sales) AS total_year_sales
    FROM weekly_sales_use w
    LEFT JOIN calendar c
    USING(week_date)
    GROUP BY c.year
)
SELECT d.demographic,
        d.year,
        d.total_demogra_sales*100/y.total_year_sales AS demogha_sales_pct
FROM demogra_sales d
LEFT JOIN year_sales y
USING(year)
ORDER BY year DESC
```
- Output:

| Row | demographic  | year | demogha_sales_pct |
|-----|--------------|------|-------------------|
| 1   | Couple       | 2020 | 28.719882877863281 |
| 2   | Family       | 2020 | 32.725289183235418 |
| 3   | unknow       | 2020 | 38.5548279389013   |
| 4   | Couple       | 2019 | 27.275156922552018 |
| 5   | Family       | 2019 | 32.474230975374169 |
| 6   | unknow       | 2019 | 40.250612102073816 |
| 7   | Couple       | 2018 | 26.38046230965961  |
| 8   | Family       | 2018 | 31.987564671761554 |
| 9   | unknow       | 2018 | 41.631973018578833 |

- **Family** customers make slightly **higher** revenue ( **4%** higher ) compared to **Couple** customers, its either because more customers have family or they just simply spent more money.
- The **unknow** demographic has the highest share of revenue -> should find more information about their demographic.

**8. Which age_band and demographic values contribute the most to Retail sales?**

```SQL
SELECT  
        s.age_band,
        s.demographic,
        ROUND(
          SUM(w.sales)*100/ 
            (SELECT SUM(sales)
             FROM weekly_sales_use
             WHERE platform = 'Retail'),2) AS retail_sale_pct
FROM weekly_sales_use w
LEFT JOIN segment_detail s
USING(segment)
WHERE w.platform = 'Retail' 
GROUP BY 
         s.age_band,
         s.demographic
ORDER BY retail_sale_pct DESC
```
- Output:

| Row | age_band      | demographic | retail_sale_pct |
|-----|---------------|--------------|-----------------|
| 1   | unknow        | unknow       | 40.52           |
| 2   | Retirees      | Family       | 16.73           |
| 3   | Retirees      | Couple       | 16.07           |
| 4   | Middle Ages   | Family       | 10.98           |
| 5   | Young Adults  | Couple       | 6.56            |
| 6   | Middle Ages   | Couple       | 4.68            |
| 7   | Young Adults  | Family       | 4.47            |

- The **Retirees** customers contribute the most for the company's revenue with the total share at about **32.8%**, especially those who have **Family** (**16.73%**)

**9. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**
- No, we cannot. Because each week has a different number of transactions, but the AVG(avg_transaction) doesn't care about the number of transaction weight.
- => We must calculate average transaction size for each year from sscratch

```SQL
SELECT  c.year,
        w.platform,
        ROUND(SUM(sales)/SUM(transactions),2) AS avg_transaction_value
FROM weekly_sales_use w
LEFT JOIN calendar c
USING(week_date)
GROUP BY c.year,
         w.platform
ORDER BY  c.year DESC,
          w.platform
```

- Output:

| Row | year | platform  | avg_transaction_value |
|-----|------|-----------|-----------------------|
| 1   | 2020 | Retail    | 36.56                 |
| 2   | 2020 | Shopify   | 179.03                |
| 3   | 2019 | Retail    | 36.83                 |
| 4   | 2019 | Shopify   | 183.36                |
| 5   | 2018 | Retail    | 36.56                 |
| 6   | 2018 | Shopify   | 192.48                |

- The average transaction value of **Shopify** is very high, almost **x5** compared to **Retail** platform, even though the share of total **revenue** made by Shopify is not high (**<3%**)
- But the avg transaction value of Shopify is **decreasing** by approximately **4.7%** each year.

=> **Shopify** has a very **high potential** for development that needs more investment and focus. 

