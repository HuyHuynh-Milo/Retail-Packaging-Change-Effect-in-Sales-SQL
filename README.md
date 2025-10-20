# 📇 Retail-Packaging-Change-Effect-in-Sales-SQL
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
- **segment_detail** table first 5 rows:

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
