# Danny's Diner - Week 1 Case Study
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

_Disclaimer: I ran my queries of the database using PostgreSQL_

## **Questions we want to solve**

* What is the total amount each customer spent at the restaurant?
* How many days has each customer visited the restaurant?
* What was/were the first item(s) from the menu purchased by each customer?
* What is the most purchased item on the menu and how many times was it purchased by all customers?
* Which item was the most popular for each customer?
* Which item was purchased by the customer after they became a member?
* Which item was purchased just before the customer became a member?
* What is the total items and amount spent for each member before they became a member?
* If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
* In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
<br></br>

## **Bonus Questions**
* Recreate the following table (see below)
* Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking of non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
<br></br>
<br></br>
## **Question 1: What is the total amount each customer spent at the restaurant?**

```postgresql
SELECT
    sales.customer_id,
    SUM(menu.price) as total_sales
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_sales DESC;
```

|customer_id|total_sales|
|-----------|-----------|
|A          |76         |
|B          |74         |
|C          |36         |

As shown in the results, the query returns that Customer A had spent $76, Customer B had spent $74, and Customer C had spent $36
<br></br>
## **Question 2: How many days has each customer visited the restaurant?**

```postgresql
SELECT
    customer_id,
    COUNT(DISTINCT order_date) AS total_visit
FROM danny_diner.sales
GROUP BY customer_id
ORDER BY total_visit DESC;
```

|customer_id|total_visit|
|-----------|-----------|
|B          |6          |
|A          |4          |
|C          |2          |

As shown in the results, Customer B visited the most with a total of 6 visits, Customer A had a total of 4 visits, while Customer C had the least amount of visits with only 2 visits.
<br></br>
## **Question 3: What was/were the first item(s) from the menu purchased by each customer?**
```postgresql
WITH ordered_sales AS (
    SELECT
        sales.customer_id,
        sales.order_date,
        RANK () OVER (
            PARTITION BY sales.customer_id
            ORDER BY sales.order_date
        ) AS order_rank,
        menu.product_name
        FROM dannys_diner.sales
        INNER JOIN dannys_diner.menu
            ON sales.product_id = menu.product_id
)
SELECT DISTINCT
    customer_id,
    product_name,
    order_date
FROM ordered_sales
WHERE order_rank = 1;
```
|customer_id|product_name|order_date|
|-----------|------------|----------|
|A          |curry       |2021-01-01|
|A          |sushi       |2021-01-01|
|B          |curry       |2021-01-01|
|C          |ramen       |2021-01-01|
As shown in the results, Customer A actually had two items purchased first as they were ordered on the same day.  Customer A had ordered curry and sushi.  Customer B had first purchased curry from the menu, while Customer C first purchased ramen.

<br></br>

## **Question 4: What is the most purchased item on the menu and how many times was it purchased by all customers?**
```postgresql
SELECT
    menu.product_name,
    COUNT(sales.customer_id) AS total_items_purchased
FROM dannys_diner.menu
INNER JOIN dannys_diner.sales
    ON menu.product_id = sales.product_id
GROUP BY menu.product_name
LIMIT 1;
```
|product_name|total_items_purchased|
|------------|---------------------|
|ramen       |8                    |

The most popular item on the menu is ramen and it was purchased by customers 8 times.
<br></br>
## **Question 5: Which item was the most popular for each customer?**
```postgresql
WITH customer_cte AS (
    SELECT
        sales.customer_id,
        menu.product_name,
        COUNT(menu.product_id) AS quantity,
        DENSE_RANK() OVER (
            PARTITION BY sales.customer_id
            ORDER BY COUNT(sales.customer_id) DESC
        ) AS item_rank
    FROM dannys_diner.sales
    INNER JOIN dannys_diner.menu
    ON menu.product_id = sales.product_id
    GROUP BY
        sales.customer_id,
        menu.product_name
)
SELECT
    customer_id,
    product_name,
    quantity
FROM customer_cte
WHERE item_rank = 1;
```
|customer_id|product_name|quantity  |
|-----------|------------|----------|
|A          |ramen       |3         |
|B          |sushi       |2         |
|B          |curry       |2         |
|B          |ramen       |2         |
|C          |ramen       |3         |

Ramen was the most popular for Customer A as this customer had ordered it 3 times.  Sushi, curry, and ramen were all popular for Customer B as they all were ordered 2 times.  Ramen was popular for Customer C as it was ordered 3 times.

## **Question 6: Which item was purchased by the customer after they became a member?**

```postgresql
WITH sales_cte AS (
    SELECT
        sales.customer_id,
        sales.order_date,
        menu.product_name,
        DENSE_RANK() OVER (
            PARTITION BY sales.customer_id
            ORDER BY sales.order_date
        )   AS order_rank
    FROM dannys_diner.sales
    INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
    INNER JOIN dannys_diner.members
        ON sales.customer_id = members.customer_id
    WHERE
        -- Note that ::DATE is postgresql syntax
        -- CAST is the equivalent syntax
        sales.order_date >= members.join_date::DATE
) 
SELECT DISTINCT
    customer_id,
    order_date,
    product_name
FROM sales_cte
WHERE order_rank = 1;
```

|customer_id|order_date|product_name|
|-----------|----------|------------|
|A          |2021-01-07|curry       |
|B          |2021-01-11|sushi       |

Customer A purchased curry right after becoming a member on 1/7/21. Customer B purchased sushi right after becoming a member on 1/11/21. Customer C is currently not a member.
<br></br>
## **Question 7: Which item was purchased just before the customer became a member?**
```postgresql
WITH prior_sales_cte AS (
    SELECT
        sales.customer_id,
        sales.order_date,
        menu.product_name,
        DENSE_RANK() OVER (
            PARTITION BY sales.customer_id
            ORDER BY sales.order_date DESC
        )   AS order_rank
    FROM dannys_diner.sales
    INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
    INNER JOIN dannys_diner.members
        ON sales.customer_id = members.customer_id
    WHERE
        sales.order_date < members.join_date::DATE
)
SELECT DISTINCT
    customer_id,
    order_date,
    product_name
FROM prior_sales_cte
WHERE order_rank = 1;
```

|customer_id|order_date|product_name|
|-----------|----------|------------|
|A          |2021-01-01|curry       |
|A          |2021-01-01|sushi       |
|B          |2021-01-04|sushi       |

Customer A had purchased curry and sushi right before becoming a member, while Customer B purchased sushi right before becoming a member.
<br></br>

## **Question 8: What is the total items and amount spent for each member before they became a member?**

```postgresql
SELECT
    sales.customer_id,
    COUNT(DISTINCT menu.product_id) AS unique_menu_items,
    SUM(menu.price) AS total_spend
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
WHERE
    sales.order_date < members.join_date::DATE
GROUP BY sales.customer_id;
```
|customer_id|unique_menu_items|total_spend|
|-----------|-----------------|-----------|
|A          |2                |25         |
|B          |2                |40         |

Before becoming a member, Customer A bought two items and spent $25. Before becoming a member, Customer B bought two items and spent $40.
<br></br>

## **Question 9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```postgresql
WITH total_points AS (
    SELECT
        sales.customer_id,
        menu.product_name,
        menu.price
        CASE
            WHEN menu.product_name = 'sushi' THEN 20 * menu.price
            ELSE 10 * menu.price
        END AS points
    FROM dannys_diner.sales
    INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
)
SELECT
    customer_id,
    SUM(points) AS sum_points
FROM total_points
GROUP BY customer id
ORDER BY customer_id;
```
|customer_id|sum_points|
|-----------|----------|
|A          |860       |
|B          |940       |
|C          |360       |

After taking into consideration the multiplier and purchases, Customer A would have a total of 860 points, Customer B would have a total of 940 points, and Customer C would have a total of 360 points.
<br></br>

## **Question 10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```postgresql
WITH total_points AS (
    SELECT
        sales.customer_id,
        menu.product_name,
        menu.price,
        CASE
            WHEN sales.order_date BETWEEN members.join_date AND (members.join_date + 6) THEN menu.price * 20
            WHEN menu.product_name = 'sushi' THEN menu.price * 20
            ELSE menu.price * 10
        END AS points
    FROM dannys_diner.sales
    INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
    INNER JOIN dannys_diner.members
        ON sales.customer_id = members.customer_id
    WHERE order_date <= '2021-01-31'
)
SELECT
    customer_id,
    SUM(points) AS sum_points
FROM total_points
GROUP BY customer_id
ORDER BY customer_id;
```

|customer_id|sum_points|
|-----------|----------|
|A          |1370      |
|B          |820       |

Customer A would have 1370 points and Customer B would have 820 points, after the special multiplier has been applied to the pricing.
<br></br>
<br></br>

## **Question 11: Recreate the following table**

_A sample of the table has been displayed below.  The table contains data of the customer id, date the customer made an order, the item(s) the customer ordered, the product price, as well noting if the customer is a member at the time of ordering.  The table has been ordered in the following way:_
* customer_id (alphabetical order)
* order_date (earliest date to latest date)
* price (largest to smallest)

|customer_id|order_date|product_name|price|member|
|-----------|----------|------------|-----|------|
|A          |2021-01-01|curry       |15   |N     |
|A          |2021-01-01|sushi       |10   |N     |
|A          |2021-01-07|curry       |15   |Y     |
|A          |2021-01-10|ramen       |12   |Y     |
|A	        |2021-01-11|ramen       |12	  |Y     |
|A	        |2021-01-11|ramen	    |12	  |Y     |
|B	        |2021-01-01|curry	    |15	  |N     |
|B	        |2021-01-02|curry	    |15	  |N     |
|B	        |2021-01-04|sushi	    |10	  |N     |
|B	        |2021-01-11|sushi	    |10	  |Y     |
|B	        |2021-01-16|ramen	    |12	  |Y     |
|B	        |2021-02-01|ramen	    |12	  |Y     |
|C	        |2021-01-01|ramen	    |12	  |N     |
|C	        |2021-01-01|ramen	    |12	  |N     |
|C	        |2021-01-07|ramen	    |12	  |N     |


```postgresql
WITH dannys_table AS (
    SELECT
        sales.customer_id,
        sales.order_date,
        sales.product_id,
        menu.product_name,
        menu.price,
        membrs.join_date
    FROM dannys_diner.sales
    INNER JOIN dannys_diner.menu
        ON sales.product_id = menu.product_id
    LEFT JOIN dannys_diner.members
        ON sales.customer_id = members.customer_id
)
SELECT
    customer_id,
    order_date,
    product_name,
    price,
    (CASE 
        WHEN order_date >= join_date THEN 'Y'
        ELSE 'N'
    END) AS member
FROM dannys_table
ORDER BY
    customer_id ASC,
    order_date ASC,
    price DESC;
```
<br></br>

## **Question 12: Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking of non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.**

```postgresql
WITH combined_sales AS (
  SELECT
    sales.customer_id,
    sales.order_date,
    menu.product_name,
    menu.price,
    CASE
      WHEN sales.order_date >= members.join_date THEN 'Y'
      ELSE 'N'
    END AS member
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
  LEFT JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
  )
  SELECT
    customer_id,
    order_date,
    product_name,
    price,
    member,
    CASE
      WHEN member = 'N' THEN NULL
      ELSE RANK() OVER (
        PARTITION BY customer_id, member
        ORDER BY order_date, price DESC
      ) END AS ranking
  FROM combined_sales;

```

|customer_id|order_date|product_name|price|member|ranking|
|-----------|----------|------------|-----|------|-------|
|A          |2021-01-01|curry       |15   |N     |null   |
|A          |2021-01-01|sushi       |10   |N     |null   |
|A          |2021-01-07|curry       |15   |Y     |1      |
|A          |2021-01-10|ramen       |12   |Y     |2      |
|A	        |2021-01-11|ramen       |12	  |Y     |3      |
|A	        |2021-01-11|ramen	    |12	  |Y     |3      |
|B	        |2021-01-01|curry	    |15	  |N     |null   |
|B	        |2021-01-02|curry	    |15	  |N     |null   |
|B	        |2021-01-04|sushi	    |10	  |N     |null   |
|B	        |2021-01-11|sushi	    |10	  |Y     |1      |
|B	        |2021-01-16|ramen	    |12	  |Y     |2      |
|B	        |2021-02-01|ramen	    |12	  |Y     |3      |
|C	        |2021-01-01|ramen	    |12	  |N     |null   |
|C	        |2021-01-01|ramen	    |12	  |N     |null   |
|C	        |2021-01-07|ramen	    |12	  |N     |null   |
