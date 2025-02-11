## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, you'll demonstrate your ability to write sophisticated SQL queries that extract valuable insights from large datasets.

The project is ideal for data analysts looking to enhance their SQL skills by working with a large-scale dataset and solving real-world business questions.

## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.

```sql
Select
	country,
	count(store_id) as total_stores
from stores
group by 1
order by count(store_id) desc
```

2. Calculate the total number of units sold by each store.

```sql
select
	st.store_name,
	st.store_id,
	sum(s.quantity) as total_sale
from sales as s
join stores as st
on
s.store_id = st.store_id
group by 1, 2
order by sum(quantity) desc
```

3. Identify how many sales occurred in December 2023.

```sql
select
count(sale_id)
from sales
where to_char (sale_date, 'MM-YYYY') = '12-2023'
```

4. Determine how many stores have never had a warranty claim filed.

```sql
select * from stores
where store_id not in
(select 
distinct store_id
from 
sales as s
right join
warranty as w
on
s.sale_id = w.sale_id)
```

5. Calculate the percentage of warranty claims marked as "Warranty Void".

```sql
select 
	Round (count(claim_id)::Numeric / (select count(*) from warranty)::Numeric * 100, 2)
	from warranty
where repair_status = 'Warranty Void'
```

6. Identify which store had the highest total units sold in the last year.

```sql
select 
	st.store_id,
	st.store_name,
	sum(s.quantity)
from sales as s 
join stores as st
on
s.store_id = st.store_id
where s.sale_date >= (current_date - Interval '1 Year')
group by 1
order by 2 desc
```

7. Count the number of unique products sold in the last year.

```sql
select 
distinct (product_id)
from sales
where sale_date >= (current_date - interval '1 Year')
```

8. Find the average price of products in each category.

```sql
select 
	p.category_id,
	c.category_name,
	avg(p.price)
from products as p
join 
category as c
on 
p.category_id = c.category_id
group by 1, 2
order by 3 desc
```

9. How many warranty claims were filed in 2020?

```sql
select 
count (*) 
from warranty
where to_char(claim_date, 'YYYY') = '2020'
```

10. For each store, identify the best-selling day based on highest quantity sold.

```sql
select * from 
(	select 
		store_id,
		to_char(sale_date,'Day'),
		sum(quantity) as total_salw,
		rank () over (partition by store_id order by sum(quantity) desc) as ranking
	from sales
	group by 1,2
) as tl

where ranking = 1
```
### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.

```sql
with Ranks
as
(select 
	st.country,
	p.product_name,
	sum(s.quantity) as total_sold,
	Rank () over (partition by st.country order by sum(s.quantity)) as ranking
from sales as s
join stores as st
on 
s.store_id = st.store_id
join products as p
on
s.product_id = p.product_id
group by 1,2
)
select * from Ranks
where ranking = 1
```

12. Calculate how many warranty claims were filed within 180 days of a product sale.

```sql
select 
count(*) as total_count
From warranty as w
left join sales as s
on 
s.sale_id = w.sale_id
where w.claim_date - s.sale_date <= 180
```

13. Determine how many warranty claims were filed for products launched in the last two years.

```sql
select
p.product_name,
count(w.claim_id) as total_calim,
count(s.sale_id) as total_sale
From warranty as w
right join sales as s
on 
s.sale_id = w.sale_id
join products as p
on 
p.product_id = s.product_id
where launch_date >= current_date - interval '2 years'
group by 1
```

14. List the months in the last three years where sales exceeded 5,000 units in the USA.
    
```sql
select 
	to_char (sale_date, 'MM-YYYY') as month,
	sum(s.quantity) as total_sold
from sales as s
join stores as st
on s.store_id = st.store_id
where st.country = 'USA'
and
s.sale_date >= current_date - interval '3 Years'
group by 1
having sum(s.quantity) > 5000
```

15. Identify the product category with the most warranty claims filed in the last two years.

```sql
Select
c.category_name,
c.category_id,
count(w.claim_id) as filled_claim
from
warranty as w
left join
sales as s
on 
w.sale_id = s.sale_id
join 
products as p
on 
p.product_id = s.product_id
join
category as c
on
p.category_id = c.category_id
where claim_date >= current_date - interval '2 years'
group by 1 , 2
order by 3 desc
```

### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
    
```sql
select
country,
total_sold,
total_claim,
Coalesce (total_claim::numeric / total_sold::numeric * 100, 0)	as risk
from 
(select 
st.country,
sum(s.quantity) as total_sold,
count(w.claim_id) as total_claim
from 
sales as s
join 
stores as st
on 
s.store_id = st.store_id
left join
warranty as w
on
s.sale_id = w.sale_id
group by 1
order by 3 desc) as t1
```

17. Analyze the year-by-year growth ratio for each store.

with Yearly_sale
as
(select 
s.store_id,
st.store_name,
extract (year from sale_date) as year,
sum(s.quantity * p.price) as total_sale
from 
sales as s
join
products  as p
on
s.product_id = p.product_id
join
stores as st
on
s.store_id = st.store_id
group by 1, 2, 3
order by 2, 3),

Growth_Ratio
as
(
select 
store_name,
year,
lag (total_sale, 1) over (partition by store_name order by year) as last_year_sale,
total_sale as current_year_sale
from Yearly_sale
)
select
store_name,
year,
last_year_sale,
current_year_sale,
Round ((current_year_sale - last_year_sale)::Numeric / last_year_sale ::Numeric * 100, 2) as Growth_Per
from Growth_Ratio
where last_year_sale is not null
and
year <> extract (year from current_date)
```

18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.

select
	case
	when p.price < 500 then 'Less Expensive'
	when p.price between 500 and 1000 then 'Medium Expensive'
	else 'Expensive'
end as Price_segment,
count (w.claim_id) as total_claimed
from 
warranty as w 
left join
sales as s
on
w.sale_id = s.sale_id
join 
products as p
on 
s.product_id = p.product_id
where claim_date >= current_date - interval '5 Years'
group by 1
```

19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.

with Paid_repaired
as
(
select 
s.store_id,
count(w.claim_id) as total_paid_rep
from 
sales as s
join warranty as w
on
s.sale_id = w.sale_id
where w.repair_status = 'Paid Repaired'
group by 1
),

Total_Repaired
as
(
select 
s.store_id,
count(w.claim_id) as total_rep
from 
sales as s
join warranty as w
on
s.sale_id = w.sale_id
group by 1
)

select 
tr.store_id,
st.store_name,
pr.total_paid_rep,
tr.total_rep,
Round(pr.total_paid_rep::Numeric / tr.total_rep::Numeric * 100,2) as ratio
from Paid_repaired as Pr
join Total_Repaired as tr
on Pr.store_id = tr.store_id
join stores as st
on tr.store_id = st.store_id
order by 4 desc
```

20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.

with Monthly_sale
as
(select
	s.store_id,
	extract (year from s.sale_date) as years,
	extract (Month from s.sale_date) as months,
	sum (p.price * s.quantity) as total_revenue
from sales as s
join products as p
on s.product_id = p.product_id
group by 1, 2, 3
)
select 
store_id,
Years,
Months,
total_revenue,
sum (total_revenue) over (partition by store_id order by years, months) as running_total
from Monthly_sale

### Bonus Question

- Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.

```sql
select
	p.product_name,
	case
	when s.sale_date between p.launch_date and p.launch_date + interval '6 Months' then '0-6 Months'
	when s.sale_date between p.launch_date + interval '6 Months'
	and p.launch_date + interval '12 Months' then '6-12 Months'
	when s.sale_date between p.launch_date + interval '12 Months'
	and p.launch_date + interval '18 Months' then '12-18 Months'
	else '18+'
	end as pls,
	sum(s.quantity) as total_sale
from sales as s 
join products as p
on s.product_id = p.product_id
group by 1, 2
order by 1, 3 desc
```

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.
