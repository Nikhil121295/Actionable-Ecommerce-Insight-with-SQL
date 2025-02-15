# Actionable-Ecommerce-Insight-with-SQL
Let's Explore the Ecommerce businesss data using SQL for the data analysis.

"Utilized SQL for in-depth data analysis, calculating order size distribution, identifying high-traffic stores, and analyzing customer behavior. Integrated demographic insights to enhance personalized marketing and guide data-driven product development."

# Try to frame the questions based on the insights we gain while exploring the data.

**Problem statement** : Analyzing sales, product, and customer data for an e-commerce company to derive insights and calculate key performance indicators (KPIs) using SQL in                           BigQuery.
**Target Matrix** :     Improve the RPC(Revenue Per Account) improve by 10 %.**
**Approach:** :         Using SQL and Tableau for data analysis and visulization.


# Let's perform basic checks on the tables to understand the data structure, data types, and the nature of the information available.
Please find the attached document for table details.

# Let's determine the order count by category, classifying orders into small, medium, and large based on their value.

with cte as (select SALES_VALUE, 
case 
  when SALES_VALUE between 0 and 9 then 'Small $0-$9.99'
  when SALES_VALUE between 10 and 20.599 then 'Medium $10-$20.99'
  when SALES_VALUE > 20 then 'Large $20+'
  end as Category
from `Ecommerce_SQL_Analysis.transaction_data`)
select Category, count(cte.SALES_VALUE) as total_orders
from cte 
where category is not null
group by category
order by 2; 

![image](https://github.com/user-attachments/assets/8f3c3aec-c07e-4f65-8c5f-241fecd35c0e)

** Insights: - A large proportion of orders fall under the "medium" range ($10-$20), indicating that customers frequently purchase items within this price bracket. This could reflect a sweet spot in pricing for most products.
**➢ Recommendation: -
1.Consider running promotions or discounts on high-value items to shift some of the medium-order customers into the large-order category.
2.Create bundling offers for small-order items to encourage customers to increase their spending.

# The number of orders that are small, medium or large order value (small:0-5 dollars, medium:5-10 dollars, large:10+

with cte as (select (SALES_VALUE * QUANTITY) as order_value, 
case 
  when (SALES_VALUE * QUANTITY) between 0 and 4 then 'Small $0-$4'
  when (SALES_VALUE * QUANTITY) between 5 and 9 then 'Medium $5-$9'
  when (SALES_VALUE * QUANTITY)> 9 then 'Large $9+'
  end as Category
from `Ecommerce_SQL_Analysis.transaction_data`)
select Category, count(order_value) as total_orders_values
from cte 
where category is not null
group by category
order by 2 desc; 

![image](https://github.com/user-attachments/assets/472bce77-a511-4fed-b316-33f04f28b979)

** Insights: - The high volume of medium and large orders highlights that customers are willing to invest more in products over $5, possibly indicating a preference for value-added or premium offerings.
➢Recommendation: - Medium and large orders dominate, so further optimize the product mix in these ranges by offering discounts, loyalty programs, or exclusive deals to retain and grow these customers.

# top 3 stores with highest foot traffic for each week (Foot traffic: number of customers transacting)
with cte as (
  select WEEK_NO,STORE_ID, count(*) customer_tras, dense_rank() over(partition by WEEK_NO order by count(*) desc) as rnk
  from `Ecommerce_SQL_Analysis.transaction_data` 
  group by 1,2
  )
select week_no,store_id,cte.customer_tras from cte where rnk <= 3 order by 3 desc , 2 desc

![image](https://github.com/user-attachments/assets/82b8034b-182a-4faa-8509-64132ae7b928)

** Insights: - The greatest number of transaction customer done is on the Store with ID 367 followed by Store ID 406 and 292

# basic customer profiling with first, last visit, number of visits,average money spent per visit and total money spent order by highest avg money

with cte as (select *, 
DATE_ADD(DATE '2000-01-01', INTERVAL Day - 1 DAY) AS converted_date
from `Ecommerce_SQL_Analysis.transaction_data` ) 
select 
  household_key, min(converted_date) as first_visit,max(converted_date) as last_visit, count(distinct converted_date) as no_of_visit,
  round(avg(quantity * sales_value),2) as customer_avg_spend_per_visit,
  round(sum(quantity * sales_value),2) as total_spendcustomer_total_spend
from cte
group by 1; 

![image](https://github.com/user-attachments/assets/5cfcd791-3819-44a3-9867-ea667c1a7d7f)

**➢Insights: -
The query is extraction the customer ID which is Household Key and what was the first visit, last visit, how many times he/she visits, the average amount he/she spends whenever he/she visits the store and the total amount he/she spends

# single customer analysis selecting most spending customer for whom we have demographic information (because not all customers in transaction data are present in demographic table) (show the demographic as well as total spent)

with most_spend_table as (
select
d.household_key,d.AGE_DESC,d.MARITAL_STATUS_CODE,d.INCOME_DESC,d.HOMEOWNER_DESC,d.HH_COMP_DESC,d.HOUSEHOLD_SIZE_DESC,d.KID_CATEGORY_DESC,
round(sum(t.SALES_VALUE)) as most_spend
from `Ecommerce_SQL_Analysis.demographic` d left join `Ecommerce_SQL_Analysis.transaction_data` t using(household_key)
group by d.household_key,d.AGE_DESC,d.MARITAL_STATUS_CODE,d.INCOME_DESC,d.HOMEOWNER_DESC,d.HH_COMP_DESC,d.HOUSEHOLD_SIZE_DESC,d.KID_CATEGORY_DESC
)
select household_key,AGE_DESC,MARITAL_STATUS_CODE,INCOME_DESC,HOMEOWNER_DESC,HH_COMP_DESC,HOUSEHOLD_SIZE_DESC,KID_CATEGORY_DESC,
most_spend 
from most_spend_table
order by most_spend desc

![image](https://github.com/user-attachments/assets/03addb4c-c23f-4e81-ba69-6a5fe9af892b)

**Insights: -
1.The customer who spent most amount is with the household key 1609 and we can see their demographics.
2.They are in the age bucket of 45-54, married.
3.Their income is between $125 - $149. They are a homeowner

# products which are most frequently bought together and the count of each combination bought together. not considering the combination twice (A-B / B-A)

select p1.SUB_COMMODITY_DESC, p2.sub_commodity_desc, count(*) pair_cnt
from `Ecommerce_SQL_Analysis.transaction_data` t1 
join `Ecommerce_SQL_Analysis.product` p1 on t1.product_id = p1.product_id
join `Ecommerce_SQL_Analysis.transaction_data` t2 on t1.BASKET_ID = t2.basket_id and t1.PRODUCT_ID < t2.PRODUCT_ID
join `Ecommerce_SQL_Analysis.product` p2 on t2.product_id = p2.product_id and p1.SUB_COMMODITY_DESC < p2.SUB_COMMODITY_DESC
group by p1.SUB_COMMODITY_DESC, p2.sub_commodity_desc
order by 3 desc

![image](https://github.com/user-attachments/assets/1ebe127d-3a47-4e0c-a3a7-078e66ea7668)

** Insights: 
1.The above query output shows which product is brought together frequently and how much time it was purchased by the customer.
2.These output we can use to increase the number of products in our inventory which is highest and medium selling

# the weekly change in Revenue Per Account (RPA) (difference in spending by each customer compared to last week)

with cte as (select household_key,WEEK_NO, round(sum(QUANTITY* sales_value),2) as weekly_spend
from `Ecommerce_SQL_Analysis.transaction_data`
group by household_key,WEEK_NO
)
select household_key,WEEK_NO,weekly_spend,lag(weekly_spend,1) over(partition by household_key order by week_no) as nextweek_values,
round((lag(weekly_spend,1) over(partition by household_key order by week_no) - cte.weekly_spend),2) as weekly_change_RPA
from cte
order by 1,2

![image](https://github.com/user-attachments/assets/4bfa9d33-67c9-4b3d-bbce-53561942afd5)

** Insights:
1.The above query shows the weekly change in revenue per account. For every customer which is household key we can see the revenue changing weekly which we can use for the inventory stocks.
2.How much revenue is generated each week, what percentage of revenue we incline or decline every week

# Recommendation:

1. Most of the products are sold within the category of small, so we need to focus of medium and large category of products as well.
2. We can combine the bundle of small with medium and small with large and then we can sell, so that the inventory will not have any expired products.
3. The top revenue we generate are from store Id 367, 406 and 292. we need to focus more on this store because they capture a large number of customers.
4. The average amount customer spent whenever they visit the store is between $ 4 - $ 5 and customer visit the store frequently.
5. The total amount a customer spent on an average is between $1500 to $2000.




