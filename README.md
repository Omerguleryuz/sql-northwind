# SQL-NORTHWIND

1) Write a solution to find the ids of products that are both low-fat and recyclable.


### EMPLOYEE ANALYSIS
-- 1) Order employees according to performances employees 
-- are above or below the calculated average

```sql
with order_counts as
(
select
	employee_id,
    count(distinct order_id) as order_count
from orders
group by employee_id
),
order_average as
(
select
	round(avg(order_count), 0) as order_average
from order_counts
)

select 
	employee_id,
    order_count,
    order_average,
    case
		when order_count >= order_average then 'Above Average' else 'Below Average' end as order_performance
from order_counts
	left join order_average
		on 1=1
order by employee_id asc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Employee%20Analysis%201.PNG)


-- 2) Order employees according to calculated revenue

```sql
with revenues_by_employees as
(
select
    orders.employee_id,
    round(sum(unit_price * quantity)) as revenue
from orders
	left join employees
		on orders.employee_id = employees.employee_id
	left join order_details
		on orders.order_id = order_details.order_id
group by orders.employee_id
),
revenue_average as
(
select
	round(avg(revenue)) as revenue_average
from revenues_by_employees
)

select 
	employee_id,
    revenue,
    revenue_average,
    case
		when revenue >= revenue_average then 'Above Average' else 'Below Average' end as revenue_performance
from revenues_by_employees
	left join revenue_average
		on 1 = 1
order by employee_id asc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Employee%20Analysis%202.PNG)


-- 3) Order counts of each employee by regions

```sql
select
	employees.employee_id,
    region.region_description,
    count(distinct orders.order_id) as order_count
from orders
	left join employees
		on orders.employee_id = employees.employee_id
	left join employeeterritories
		on employees.employee_id = employeeterritories.employee_id
	left join territories 
		on employeeterritories.territory_id = territories.territory_id
	left join region
		on territories.region_id = region.region_id
group by
	region.region_description,
    employees.employee_id;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Employee%20Analysis%203.PNG)
    
-- 4) Loyal customer count of each employee
-- Loyal customer: A customer who purchased at least 3 times 
-- from the same employee

```sql
with customer_counts as 
(
select
	employee_id,
    customer_id,
    count(customer_id) as customer_count
from orders
group by 
	employee_id,
    customer_id
having count(customer_id) >= 3
order by employee_id asc
)

select
	employee_id,
	count(employee_id) as loyal_customer_count
from customer_counts
group by employee_id
order by count(employee_id) desc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Employee%20Analysis%204.PNG)


-- SUPPLIER ANALYSIS

-- 1) From which countries our supplies come from?

```sql
select
	suppliers.country,
    count(distinct orders.order_id) as order_count
from orders
	left join order_details
		on orders.order_id = order_details.order_id
	left join products
		on order_details.product_id = products.product_id
	left join suppliers
		on products.supplier_id = suppliers.supplier_id
where country is not null
group by suppliers.country
order by order_count desc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Supplier%20Analysis%201.PNG)


-- 2) What is our most supplied product?

```sql
select
	products.product_id,
    count(distinct orders.order_id) as supply_count
from orders
	left join order_details
		on orders.order_id = order_details.order_id
	left join products
		on order_details.product_id = products.product_id
	left join suppliers
		on products.supplier_id = suppliers.supplier_id
where suppliers.country is not null
group by 
	products.product_id;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Supplier%20Analysis%202.PNG)


-- 3) Monthly distribution of our supplied products 

```sql
select
    extract(month from orders.order_date) ASC months,
    count(products.product_id) as supply_count
from orders
    left join order_details
        on orders.order_id = order_details.order_id
    left join products
        on order_details.product_id = products.product_id
group by months
order by months asc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Supplier%20Analysis%203.PNG)


-- CUSTOMER ANALYSIS

-- 1) Which product category gets the most demand


```sql
select
	products.category_id,
    count(distinct orders.order_id) as order_count
from orders
	left join order_details
		on orders.order_id = order_details.order_id
	left join products
		on order_details.product_id = products.product_id
where category_id is not null
group by products.category_id
order by order_count desc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Customer%20Analysis%201.PNG)


-- 2) From which product category we get the most revenue? 

```sql
select
	category_id,
    round(sum(order_details.quantity * order_details.unit_price)) as revenue
from orders
	left join order_details
		on orders.order_id = order_details.order_id
	left join products
		on order_details.product_id = products.product_id
where 
	category_id is not null
group by
	category_id
order by revenue desc;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Customer%20Analysis%202.PNG)


-- 3) Monthly distribution of order quantites of product categories 

```sql
select	
    category_id,
    extract(month from orders.order_date) as month,
    sum(order_details.quantity) as order_quantity
from orders
    left join order_details
        on orders.order_id = order_details.order_id
    left join products
        on order_details.product_id = products.product_id
where 
    category_id is not null
group by
    category_id,
    extract(month from orders.order_date)
order by 
    extract(month from orders.order_date),
    category_id;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Customer%20Analysis%203.PNG)


-- 4)  Monthly distribution of revenues of product categories 


```sql
select
    category_id,
    extract(month from orders.order_date) as month,
    ROUND(SUM(order_details.quantity * order_details.unit_price)) as revenue
from orders
    left join order_details
        on orders.order_id = order_details.order_id
    left join products
        on order_details.product_id = products.product_id
where 
    category_id is not null
group by
    category_id,
    extract(month from orders.order_date)
order by 
    extract(month from orders.order_date),
    category_id;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Customer%20Analysis%204.PNG)

-- ORDERS ANALYSIS

-- 1) Percentage of on time orders 


```sql
with on_time_orders_table as
(
select
	order_id,
    required_date,
    shipped_date,
	count(distinct order_id) as on_time_orders
from orders
where required_date <= shipped_date
group by 
	order_id,
    required_date,
    shipped_date
)

select
	count(distinct on_time_orders_table.order_id) as on_time_orders,
    count(distinct orders.order_id) as total_orders,
    round(count(distinct on_time_orders_table.order_id) / count(distinct orders.order_id) * 100, 2) as percentage
from orders
	left join on_time_orders_table
		on orders.order_id = on_time_orders_table.order_id;
```

![alt text](https://github.com/Omerguleryuz/sql-northwind/blob/main/Screenshot/Customer%20Analysis%205.PNG)


