Given this ERD

<img src="https://github.com/T1mSchneider/T1mSchneider.github.io/blob/master/images/SQL2.png"/>


Show the employee's first_name and last_name, a "num_orders" column with a count of the orders taken, and a column called "Shipped" that displays "On Time" if the order shipped_date is less or equal to the required_date, "Late" if the order shipped late, "Not Shipped" if shipped_date is null.
Order by employee last_name, then by first_name, and then descending by number of orders.

```sql
select
e.first_name,
e.last_name,
count(o.order_id) as num_orders,
case
	when shipped_date <= required_date then 'On Time'
	when shipped_date is null then 'Not Shipped'
	else 'Late'
end as shipped
from employees e
join orders o on e.employee_id = o.employee_id
group by first_name, last_name, shipped
order by last_name, first_name, num_orders desc
```
Show how much money the company lost due to giving discounts each year, order the years from most recent to least recent. Round to 2 decimal places

```sql
select
	year(o.order_date) as order_year,
	round(sum((od.discount)* (od.quantity *  p.unit_price)),2) as discount_amount
from orders o
join order_details od on o.order_id = od.order_id
join products p on p.product_id = od.product_id
group by order_year
order by order_year desc
```
