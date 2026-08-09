-- change over time trends
-- Analyze sales performance over time

use datawarehouseanalytics;
select
 year(order_date) as order_year,
month(order_date) as order_month,
sum(sales_amount)  as total_sales,
count(distinct customer_key) as total_customers,
sum(quantity) as total_quantity
from fact_sales 
where order_date is not null
group by year(order_date),month(order_date)
order by year(order_date),month(order_date);


-- cumulativer analysis 
-- aggregate the data progressively over time
-- whether buisness is growing or declining 
-- calculate the total sales per month and running total of sales over time

select 
order_year,
total_sales,
sum(total_sales) over(  order by order_year) as running_total,
round(avg(avg_sales) over(  order by order_year),2) as moving_avg
from
(
select 
 year(order_date) as order_year,
month(order_date) month_sales,
sum(sales_amount) as total_sales,
avg(sales_amount) as avg_sales
from fact_sales
where order_date is not null
group by month(order_date),year(order_date)
)t;


/*performance analysis;
 Analyze the yearly performance of products by comparing
each products sales of both its average sales performance and the previous years sales*/

with yearly_product_sales as (
select  year(order_date) as order_year,
product_name,
sum(sales_amount) as current_sales
from fact_sales  as f
left join dim_products  as d
on f.product_key=d.product_key
where order_date is not null
group by year(order_date),product_name
order by year(order_date),product_name)
 select 
 order_year,product_name,current_sales,
 avg(current_sales) over(partition by product_name) as avg_sales,
 current_sales-avg(current_sales) over(partition by product_name) as diff_avg,
 case when current_sales-avg(current_sales) over(partition by product_name) >0 then "above avg"
      when  current_sales-avg(current_sales) over(partition by product_name) <0 then "below avg"
  else "avg"
   end as avg_change,
   -- year over  year analysis
 lag(current_sales) over(partition by product_name order by order_year) as previous_sales  ,
 current_sales- lag(current_sales) over(partition by product_name order by order_year)  as diff_previous,
 case when current_sales- lag(current_sales) over(partition by product_name order by order_year) >0 then "increase"
      when  current_sales- lag(current_sales) over(partition by product_name order by order_year) <0 then "decrease"
  else "no change"
   end as py_sales_change
 from yearly_product_sales
 order by product_name,order_year;
 
 -- part to whole analysis
 -- which categories contribute the most to overall sales ?
 with category_sales as(
 select 
 category,
 sum(sales_amount) as total_sales
 from fact_sales as f
left join dim_products as d 
on f.product_key=d.product_key
 group by category)
 select category,total_sales,
 sum(total_sales) over() as overall_sales,
concat( total_sales/sum(total_sales) over() * 100,"%")  as percentage_total
 from category_sales;
 
 
 /* segment products into cost ranges and count how many products fall into each segment */
 with product_segments as (
 select  
 product_key,
 product_name,
 cost,
 case when cost < 100 then "below 100"
      when cost between 100 and 500 then "100-500"
      when cost between 500 and 1000 then "500-1000"
      else "above 1000"
end cost_range       
 from dim_products)
 select 
 cost_range,
 count(product_key) as total_products
 from product_segments
 group by cost_range
 order by total_products desc;
 
/* Group customers into three segments based on their spending behavior:  
   - VIP: Customers with at least 12 months of history and spending more than 5,000.  
   - Regular: Customers with at least 12 months of history but spending 5,000 or less.  
   - New: Customers with a lifespan less than 12 months.  
   And find the total number of customers by each group  
*/


 with customer_spending as (
    select
        f.customer_key,
        sum(sales_amount) as total_spending,
        min(order_date) as first_order,
        max(order_date) as last_order,
        timestampdiff(month, min(order_date), max(order_date)) as lifespan
    from fact_sales as f
    left join dim_customers as d
    on f.customer_key = d.customer_key
    group by f.customer_key
)
select
    customer_segment,
    count(*) as total_customers
from (
    select
        customer_key,
        case 
            when lifespan >= 12 and total_spending > 5000 then "VIP"
            when lifespan >= 12 and total_spending <= 5000 then "REGULAR"
            else "NEW"
        end as customer_segment
    from customer_spending
) t
group by customer_segment
order by total_customers desc;


/*
Customer Report
Purpose:
 - This report consolidates key customer metrics and behaviors

Highlights:
1. Gathers essential fields such as names, ages, and transaction details.
2. Segments customers into categories (VIP, Regular, New) and age groups.
3. Aggregates customer-level metrics:
   - total orders
   - total sales
   - total quantity purchased
   - total products
   - lifespan (in months)
4. Calculates valuable KPIs:
   - recency (months since last order)
   - average order value
   - average monthly spend
*/
/*base query= retrieves core columns from tables */

create view report_customers2 as 
with base_query as (
select 
f.order_number,
f.product_key,
f.order_date,
f.sales_amount,
f.quantity,
d.customer_key,
d.customer_number,
concat(d.first_name," ",d.last_name) as full_name,
timestampdiff(year,birthdate,now()) as age
from fact_sales as f
inner join dim_customers as d
on f.customer_key=d.customer_key
where order_date is not null)


, customer_aggregation as (
/* customer aggregation: summarizes key metrices at the customer level */
select 
customer_key,
customer_number,
full_name,
age,
count(distinct order_number) as total_orders,
sum(sales_amount) as total_quantity,
count(distinct product_key) as total_products,
max(order_date)  as last_order_date,
timestampdiff(month,min(order_date),max(order_date)) as life_span
from base_query
group by  customer_key,
customer_number,
full_name,
age)
select 
customer_key,
customer_number,
full_name,
age,
case when age < 20 then "under 20"
     when age between 20 and 29 then "20-29"
     when age between 30 and 39 then "30-39"
     when age between 40 and 49 then "40-49"
 else "50 and above "
 end as age_group ,
 case when life_span >= 12 and total_quantity > 5000 then "VIP"
      when life_span >=12 and total_quantity <=  5000 then "redgular"
      else "new"
end as customer_segment,
total_orders,
total_quantity,
total_products,
last_order_date,
timestampdiff(month,last_order_date,now()) as recency,

--  compute avg order values

total_quantity/total_orders as avg_order_value,

-- compute avg monthly spend 
case when life_span =0 then total_quantity
else total_quantity/life_span
end as avg_monthly_spend
from customer_aggregation;
select * from report_customers2;se
