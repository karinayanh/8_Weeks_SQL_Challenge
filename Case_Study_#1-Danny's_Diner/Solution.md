#  **Case Study #1: Danny's Diner**

<img width="352" height="347" alt="image" src="https://github.com/user-attachments/assets/51adb461-f672-4302-b975-b9f9512b68fe" />

The entire case can be viewed [here](https://8weeksqlchallenge.com/case-study-1/).

## **Introduction**

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

## **Problem Statement**

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers. He plans on using these insights to help him decide whether he should expand the existing customer loyalty program

Danny has provided a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough to write fully functioning SQL queries to help him answer his questions. He has shared the following datasets for this case study with the following columns: 

- sales: customer_id, order_date, product_id 
- menu: product_id, product_name, price
- members: customer_id, join_date

<img width="333" height="167" alt="image" src="https://github.com/user-attachments/assets/5bdaed16-7372-47f8-a249-7dd99412936c" />

## **Case Study Questions**

The following questions were answered using DB Fiddle (PostgrSQL v13) 

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
  	sales.customer_id as Customer,
    SUM(menu.price) as Total_Sales
FROM dannys_diner.menu
INNER JOIN dannys_diner.sales ON menu.product_id = sales.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
````
The total amount each customer spent at the restaurant is: 

| Customer    | Total_Sales  | 
|:-----------:|:------------:|
|     A       |     76       |
|     B       |     74       |
|     C       |     36       |

**2. How many days has each customer visited the restaurant?**

````sql
SELECT
  	sales.customer_id as Customer,
    count(distinct sales.order_date) as Total_Visits
FROM dannys_diner.sales
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
````
The total days each customer visited is as follow:  

| Customer    | Total_Visits | 
|:-----------:|:------------:|
|     A       |     4        |
|     B       |     6        |
|     C       |     2        |

**3. What was the first item from the menu purchased by each customer?**
````sql
WITH sort_sales AS (
SELECT
  	sales.customer_id,
    sales.order_date, 
    menu.product_name,
    DENSE_RANK () OVER (
    	PARTITION by sales.customer_id
   	 	ORDER BY sales.order_date) AS sort  
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)
    
SELECT 
	customer_id,
	order_date,
    product_name
FROM sort_sales
WHERE sort = 1 
GROUP BY customer_id, order_date, product_name;
````
The following table shows the first item that was purchased by each customer on 2021-01-01:

| customer_id    | order_date   |  product_name | 
|:--------------:|:------------:|:-------------:|
|     A          | 2021-01-01   |   Curry       |
|     A          | 2021-01-01   |   Sushi       |
|     B          | 2021-01-01   |   Curry       |
|     C          | 2021-01-01   |   Ramen       |

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT 
	menu.product_name AS Product_Name,
    COUNT (sales.product_id) AS most_purchased
FROM dannys_diner.sales 
INNER JOIN dannys_diner.menu 
	ON sales.product_id = menu.product_id
GROUP BY menu.product_name 
ORDER BY most_purchased DESC
LIMIT 1; 
````
The most purchased item is Ramen. The table below depicts the number of times it was purchased since 2021-01-01: 

| product_name   | most_purchased | 
|:--------------:|:--------------:|
|     Ramen      |     8          |

If we want to have a breakdown of how many items each customer bought the most popular item since 2021-01-01. The SQL code will be:

````sql
With Most_purchased AS (
  SELECT menu.product_name,
  		 COUNT(menu.product_name) AS purchase_times
  FROM dannys_diner.menu 
  INNER JOIN dannys_diner.sales 
  ON sales.product_id = menu.product_id 
  GROUP BY menu.product_name 
  ORDER BY COUNT(menu.product_name) DESC
  LIMIT 1 )
  
SELECT customer_id, menu.product_name, COUNT(menu.product_name) AS purchased_times 
FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu ON menu.product_id = sales.product_id
  INNER JOIN Most_purchased ON Most_purchased.product_name = menu.product_name 
  GROUP BY customer_id, menu.product_name; 
````
| Customer| product_name   | most_purchased | 
|:-------:|:--------------:|:--------------:|
|    A    |     Ramen      |     3          |
|    B    |     Ramen      |     2          |
|    C    |     Ramen      |     3          |

**5. Which item was the most popular for each customer?**

````sql
WITH most_popular AS (
SELECT 
	sales.customer_id,
  	menu.product_name,
    COUNT (menu.product_id) AS most_purchased,
  	DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
FROM dannys_diner.sales 
INNER JOIN dannys_diner.menu 
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id, menu.product_name
  )
  
SELECT 
	customer_id, 
    product_name, 
    most_purchased
FROM most_popular
WHERE rank = 1; 
````
The table below shows the most purchased products per customer. One customer may have more than one popular item.  

| customer_id   | product_name | most_purchased |
|:-------------:|:------------:|:--------------:|
|     A         |     Ramen    | 3              |
|     B         |     Ramen    | 3              |
|     B         |     Curry    | 2              |
|     B         |     Sushi    | 2              |
|     C         |     Ramen    | 3              |

**6. Which item was purchased first by the customer after they became a member?**

````sql
WITH after_membership AS (
SELECT 
	members.customer_id,
  	sales.product_id,
    ROW_NUMBER() OVER(
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row
FROM dannys_diner.members 
INNER JOIN dannys_diner.sales 
	ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
  )
  
SELECT 
	customer_id, 
    product_name
FROM after_membership
INNER JOIN dannys_diner.menu 
	ON after_membership.product_id = menu.product_id
WHERE row = 1
ORDER BY customer_id ASC;

````
Customer A and B are the only ones with membership. The first item that customer A bought was Ramen and the first item that Customer B bought was Sushi. 

| customer_id    | product_name   | 
|:--------------:|:--------------:|
|     A          |     Ramen      |
|     B          |     Sushi      |

**7. Which item was purchased just before the customer became a member?**

````sql
WITH before_membership AS (
SELECT 
	members.customer_id,
  	sales.product_id,
    ROW_NUMBER() OVER(
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS row
FROM dannys_diner.members 
INNER JOIN dannys_diner.sales 
	ON members.customer_id = sales.customer_id
    AND sales.order_date < members.join_date
  )
  
SELECT 
	customer_id, 
    product_name
FROM before_membership
INNER JOIN dannys_diner.menu 
	ON before_membership.product_id = menu.product_id
WHERE row = 1
ORDER BY customer_id ASC; 
````
The last item that customer A bought before joining as member was Ramen and the last item that Customer B bought before joining as member was Sushi. 

| customer_id    | product_name   | 
|:--------------:|:--------------:|
|     A          |     Ramen      |
|     B          |     Sushi      |

**8. What is the total items and amount spent for each member before they became a member?**

````sql
SELECT
	sales.customer_id, 
    COUNT(sales.product_id) AS total_items, 
    SUM(menu.price) AS total_sales 
FROM dannys_diner.sales 
INNER JOIN dannys_diner.members 
	ON sales.customer_id = members.customer_id
    AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
GROUP BY sales.customer_id 
ORDER BY sales.customer_id ASC; 
````
Customer A bought a total of 2 items before they became a member amounting a total of $25 dollars. 
Customer B bought a total of 3 items before they became a member amounting a total of $40 dollars. 

| customer_id   | total_items  | total_sales    |
|:-------------:|:------------:|:--------------:|
|     A         |     2        | 25             |
|     B         |     3        | 40             |

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

````sql
WITH product_points AS ( 
  SELECT
  	menu.product_id, 
  CASE
  	WHEN product_id = 1 THEN price * 20
  	ELSE price * 10 END as points
  FROM dannys_diner.menu 
)

SELECT 
	sales.customer_id, 
    SUM(product_points.points) as total_points
FROM dannys_diner.sales
INNER JOIN product_points
	ON sales.product_id = product_points.product_id
GROUP BY sales.customer_id 
ORDER BY sales.customer_id ASC;
````
The total points for each customer is as follow: 
- Customer A: 860 
- Customer B: 940
- Customer C: 360

| customer_id    | total_points   | 
|:--------------:|:--------------:|
|     A          |     860        |
|     B          |     940        |
|     C          |     360        |


**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

````sql
WITH week_after_join AS(
  SELECT 
  	customer_id, 
    join_date,
    join_date + 6 AS week_later,
    DATE_TRUNC(
        'month', '2021-01-31'::DATE)
        + interval '1 month' 
        - interval '1 day' AS last_date
  FROM dannys_diner.members 
  )
  
SELECT
 	sales.customer_id, 
    SUM(CASE 
        WHEN sales.order_date BETWEEN join_date AND week_after_join.week_later THEN price * 20
        WHEN sales.order_date NOT BETWEEN join_date AND week_after_join.week_later AND menu.product_name = 'sushi' THEN price *20 
        ELSE menu.price *10 
        END) AS points
FROM dannys_diner.sales 
INNER JOIN week_after_join 
 	ON sales.customer_id = week_after_join.customer_id
INNER JOIN dannys_diner.menu
	 ON menu.product_id = sales.product_id
 	AND week_after_join.join_date <= sales.order_date 
 	AND sales.order_date <= week_after_join.last_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC; 
````
Customer A has 1020 points at the end of the first week of joining the program, and customer B has 320 points after the first week of joining the program. 

| customer_id    | points    | 
|:--------------:|:---------:|
|     A          |     1020  |
|     B          |     320   |

## **Bonus Questions**
**Join All The Things**
Recreate the following table output using the available data:

<img width="286" height="353" alt="image" src="https://github.com/user-attachments/assets/7f33bcbd-ae6b-436f-bcb5-46816575ac96" />

Answer:
````sql
SELECT 
	sales.customer_id, 
    sales.order_date, 
    menu.product_name, 
    menu.price, 
    CASE
      WHEN members.join_date > sales.order_date THEN 'N'
      WHEN members.join_date <= sales.order_date THEN 'Y' 
      ELSE 'N'
      END AS member
      
FROM dannys_diner.sales 
LEFT JOIN dannys_diner.members 
	ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
ORDER BY members.customer_id, sales.order_date
````
| customer_id	| order_date	| product_name	| price |	member | 
|:-----------:|:-----------:|:-------------:|:-----:|:------:|
|A            |	2021-01-01  |	curry	        |15     |	N      |
|A          	|2021-01-01	  |sushi	        |10     |	N      |
|A	          |2021-01-07  	|curry	        |15     |	Y      |
|A	          |2021-01-10 	|ramen	        |12     | Y      |
|A	          |2021-01-11	  |ramen          |	12    |	Y      |
|A	          |2021-01-11 	|ramen	        |12     |	Y      |
|B	          |2021-01-01	  |curry	        |15     |	N      |
|B	          |2021-01-02	  |curry	        | 15    |	N      |
|B	          |2021-01-04	  |sushi          |	10    |	N      |
|B	          |2021-01-11	  |sushi	        |10     |	Y      |
|B	          |2021-01-16 	|ramen	        |12     |	Y      |
|B	          |2021-02-01	  |ramen	        |12     |	Y      |
|C	          |2021-01-01	  | ramen	        |12     |	N      |
|C	          |2021-01-01	  |ramen	        |12     |	N      |
|C            | 2021-01-07	|ramen	        |12     |	N      |


**Rank All The Things**
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

<img width="331" height="354" alt="image" src="https://github.com/user-attachments/assets/4bd5a479-05ff-43f1-8813-e64fb6f2251c" />

Answer:
````sql
WITH dannys_data AS (
SELECT 
	sales.customer_id, 
    sales.order_date, 
    menu.product_name, 
    menu.price, 
    CASE
      WHEN members.join_date > sales.order_date THEN 'N'
      WHEN members.join_date <= sales.order_date THEN 'Y' 
      ELSE 'N'
      END AS member
      
FROM dannys_diner.sales 
LEFT JOIN dannys_diner.members 
	ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
	ON sales.product_id = menu.product_id
)

SELECT * , 
	CASE 
    	WHEN member = 'N' then NULL
        ELSE RANK() OVER(
          PARTITION by customer_id, member 
          ORDER BY customer_id, order_date
          )
  		END as rank
FROM dannys_data; 
````

| customer_id	| order_date	| product_name	| price |	member | ranking |
|:-----------:|:-----------:|:-------------:|:-----:|:------:|:-------:|
|A            |	2021-01-01  |	curry	        |15     |	N      |  null   |
|A          	|2021-01-01	  |sushi	        |10     |	N      |  null   |
|A	          |2021-01-07  	|curry	        |15     |	Y      |   1     |
|A	          |2021-01-10 	|ramen	        |12     | Y      |   2     |
|A	          |2021-01-11	  |ramen          |	12    |	Y      |   3     |
|A	          |2021-01-11 	|ramen	        |12     |	Y      |   3     |
|B	          |2021-01-01	  |curry	        |15     |	N      |  null   |
|B	          |2021-01-02	  |curry	        | 15    |	N      |  null   |
|B	          |2021-01-04	  |sushi          |	10    |	N      |  null   |
|B	          |2021-01-11	  |sushi	        |10     |	Y      |  1      |
|B	          |2021-01-16 	|ramen	        |12     |	Y      |  2      |
|B	          |2021-02-01	  |ramen	        |12     |	Y      |  3      | 
|C	          |2021-01-01	  | ramen	        |12     |	N      |  null   |
|C	          |2021-01-01	  |ramen	        |12     |	N      |  null   |
|C            | 2021-01-07	|ramen	        |12     |	N      |  null   |
