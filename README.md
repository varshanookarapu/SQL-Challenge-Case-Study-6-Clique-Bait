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
```

**Question 3:** What is the unique number of visits by all users per month?
```sql
```

**Question 4:** What is the number of events for each event type?
```sql
SELECT event_type, COUNT(*) as event_count FROM clique_bait.events 
GROUP BY event_type
ORDER BY event_type
```
<img width="1015" height="282" alt="image" src="https://github.com/user-attachments/assets/97f33df7-5707-4120-b2b3-82d596b1f39e" />

**Question 5:** What is the percentage of visits which have a purchase event?

```sql
```
**Question 6:** What is the percentage of visits which view the checkout page but do not have a purchase event?
```sql
```

**Question 7:** What are the top 3 pages by number of views?

```sql
```
**Question 8:** What is the number of views and cart adds for each product category?

```sql
```

**Question 9:** What are the top 3 products by purchases?

```sql
```
