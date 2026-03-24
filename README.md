# Case Study 6 : Clique Bait

## 1. Enterprise Relationship Diagram
<img width="1842" height="788" alt="image" src="https://github.com/user-attachments/assets/8286ba3a-f9d2-4980-91c9-d24f19f5d9d8" />

## 2. Digital Analysis

**Question 1:** How many users are there?

```sql
SELECT COUNT(DISTINCT user_id) AS user_count FROM clique_bait.users;
```
<img width="244" height="106" alt="image" src="https://github.com/user-attachments/assets/65552fc1-a738-44c4-9eab-f7381a68085c" />

**Question 2:** How many cookies does each user have on average?
```sql
SELECT  ROUND((COUNT(cookie_id):: NUMERIC /(SELECT COUNT(DISTINCT user_id) FROM clique_bait.users)::NUMERIC),2)   as avegrage_cookie_count FROM clique_bait.users
```
<img width="286" height="112" alt="image" src="https://github.com/user-attachments/assets/6941c1d0-933c-4deb-8ba2-306531acc0c6" />

**Question 3:** What is the unique number of visits by all users per month?

```sql
SELECT EXTRACT( 'month' FROM event_time) as month,TO_CHAR(event_time,'Month') as month_name, COUNT(DISTINCT visit_id)  as unique_visit_count
FROM clique_bait.events e LEFT JOIN clique_bait.users u ON
e.cookie_id = u.cookie_id
GROUP BY EXTRACT( 'month' FROM event_time),TO_CHAR(event_time,'Month')
ORDER BY EXTRACT( 'month' FROM event_time)
```
<img width="1130" height="273" alt="image" src="https://github.com/user-attachments/assets/1e18f260-7921-49c1-ba06-e571330dcf70" />


**Question 4:** What is the number of events for each event type?
```sql
SELECT event_type, COUNT(*) as event_count FROM clique_bait.events 
GROUP BY event_type
ORDER BY event_type
```
<img width="1015" height="282" alt="image" src="https://github.com/user-attachments/assets/97f33df7-5707-4120-b2b3-82d596b1f39e" />

**Question 5:** What is the percentage of visits which have a purchase event?

```sql
SELECT  100*COUNT(CASE WHEN event_type=3 THEN visit_id END)/ (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events ) :: NUMERIC AS percentage_of_visits_for_purchase_event

FROM
clique_bait.events 

```
<img width="385" height="104" alt="image" src="https://github.com/user-attachments/assets/fda7daed-b308-4da3-b087-f9f42120cf34" />


**Question 6:** What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
```

**Question 7:** What are the top 3 pages by number of views?

```sql
SELECT  e.page_id ,page_name ,COUNT(visit_id) visitor_count ,
RANK() OVER( ORDER BY COUNT(visit_id) DESC) as rank
FROM clique_bait.events e LEFT JOIN clique_bait.page_hierarchy ph ON
e.page_id = ph.page_id
GROUP BY e.page_id,page_name
LIMIT 3
```
<img width="1528" height="191" alt="image" src="https://github.com/user-attachments/assets/0ba58084-267b-4a63-8fd3-ede639fc9092" />

**Question 8:** What is the number of views and cart adds for each product category?

```sql
SELECT  product_category,

COUNT(CASE WHEN e.event_type = 1  THEN 1 END) AS page_view_count, 
COUNT(CASE WHEN e.event_type = 2  THEN 1 END) AS cart_add_count

FROM clique_bait.events e LEFT JOIN clique_bait.page_hierarchy ph 
ON e.page_id = ph.page_id
LEFT JOIN clique_bait.event_identifier ei 
ON e.event_type=ei.event_type
WHERE product_category IS NOT NULL
GROUP BY product_category 

```
<img width="1485" height="198" alt="image" src="https://github.com/user-attachments/assets/43526b7f-ea3e-4f9d-aa2a-4cc19b0a2104" />


**Question 9:** What are the top 3 products by purchases?

```sql
--in this cte we get the distinct visit_id which led to the purchase event 
WITH cte AS
(
SELECT  DISTINCT visit_id  FROM clique_bait.events WHERE event_type =3
) 

-- From the cte we are querying page name and counting the visit ids by joing the events and cte together and grouping by pagename

SELECT page_name, COUNT(e.visit_id) as visit_count 
FROM clique_bait.events e
LEFT JOIN clique_bait.page_hierarchy ph ON
e.page_id = ph.page_id
LEFT JOIN cte ON
e.visit_id = cte.visit_id
WHERE ph.product_id IS NOT NULL
GROUP BY page_name
ORDER BY COUNT(e.visit_id) DESC
LIMIT 3
```
<img width="1238" height="239" alt="image" src="https://github.com/user-attachments/assets/a5e10d7a-c81f-406f-abb9-3685a1822604" />

---

## 2. Funnel Analysis

```sql
-- -- Product View Count and add to cart count 

WITH table1 AS
(
SELECT product_id, page_name as product, visit_id,

SUM(CASE WHEN event_type =1 THEN 1 ELSE 0 END) AS page_view_count ,
SUM(CASE WHEN event_type =2 THEN 1 ELSE 0 END) AS addtocart_count

FROM  clique_bait.events e LEFT JOIN
clique_bait.page_hierarchy ph ON
e.page_id = ph.page_id
WHERE  product_id IS NOT NULL
GROUP BY e.page_id,page_name , product_id,visit_id
ORDER BY product_id
),

-- purchase events 

table2 AS (
  
  SELECT DISTINCT visit_id FROM clique_bait.events WHERE event_type = 3
),

table3 AS
(
SELECT product_id,product,t1.visit_id,page_view_count,addtocart_count, CASE WHEN t1.visit_id IS NOT NULL THEN 1 ELSE 0 END as purchase_count
FROM table1 t1 LEFT JOIN table2 t2 ON
t1.visit_id = t2.visit_id
)
```
