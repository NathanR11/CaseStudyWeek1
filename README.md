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

## **Entity Relational Diagram**

<img width="803" alt="ERD" src="https://user-images.githubusercontent.com/75928570/196569691-25bd3478-dbb4-4e7d-9bcf-9897d7ff284c.png">
<br></br>

## **Table 1: sales**
<img width="264" alt="Screen Shot 2022-10-18 at 5 31 38 PM" src="https://user-images.githubusercontent.com/75928570/196569801-bd8e8309-eeac-4dc9-9dde-906e599e6ec6.png">
<br></br>

## **Table 2: menu**
<img width="231" alt="Screen Shot 2022-10-18 at 5 32 24 PM" src="https://user-images.githubusercontent.com/75928570/196569884-8174b3b6-eff0-41c7-a77b-b18a29366462.png">
<br></br>

## **Table 3: members**

<img width="192" alt="Screen Shot 2022-10-18 at 5 33 49 PM" src="https://user-images.githubusercontent.com/75928570/196570016-2cc60087-bdd2-461d-af68-7058bc079700.png">
<br></br>

## Schema for Database Creation

_The following query was used in PostgreSQL to create the database with necessary schema, tables, and datasets_

```postgresql
CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```
