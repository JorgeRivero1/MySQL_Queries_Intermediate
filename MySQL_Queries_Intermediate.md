
# 🐬 MySQL Exercises: Querying Data



## Apple Product Counts


We’re analyzing user data to understand how popular Apple devices are among users who have performed at least one event on the platform. Specifically, we want to measure this popularity across different languages. Count the number of distinct users using Apple devices —limited to "macbook pro", "iphone 5s", and "ipad air" — and compare it to the total number of users per language.

Present the results with the language, the number of Apple users, and the total number of users for each language. Finally, sort the results so that languages with the highest total user count appear first.

playbook_events

| Columna      | Tipo de dato |
|--------------|--------------|
| device       | text         |
| event_name   | text         |
| event_type   | text         |
| location     | text         |
| occurred_at  | timestamp    |
| user_id      | bigint       |

playbook_users

| Columna       | Tipo de dato   |
|---------------|----------------|
| activated_at  | date           |
| company_id    | bigint         |
| created_at    | datetime       |
| language      | text           |
| state         | text           |
| user_id       | bigint         |




 ```sql
with cte as (
    SELECT *
    FROM playbook_events pe
    JOIN playbook_users pu USING(user_id)
),
cte_total as (
    SELECT language, count(DISTINCT user_id) as conteo_total
    FROM cte
    GROUP BY language
),

cte_apple as(
    SELECT DISTINCT user_id, language, count(DISTINCT user_id) as conteo_apple
    FROM cte
    WHERE device LIKE '%macbook pro%' 
        or device LIKE '%iphone 5s%' 
        or device LIKE '%ipad air%'
    GROUP BY language
)


SELECT ct.language, 
    COALESCE(conteo_apple,0) AS conteo_apple, 
    conteo_total
FROM cte_total ct
LEFT JOIN cte_apple ca on ct.language = ca.language
ORDER BY conteo_total DESC


 ```

## Titanic Survivors and Non-Survivors

Make a report showing the number of survivors and non-survivors by passenger class. Classes are categorized based on the pclass value as:

•	First class: pclass = 1

•	Second class: pclass = 2

•	Third class: pclass = 3

Output the number of survivors and non-survivors by each class.

| Columna      | Tipo de dato |
|--------------|--------------|
| age          | double       |
| cabin        | text         |
| embarked     | text         |
| fare         | double       |
| name         | text         |
| parch        | bigint       |
| passengerid  | bigint       |
| pclass       | bigint       |
| sex          | text         |
| sibsp        | bigint       |
| survived     | bigint       |
| ticket       | text         |


```sql

SELECT survived, 
        SUM(CASE WHEN pclass = 1 then 1 else 0 end) as first_class,
        SUM(CASE WHEN pclass = 2 then 1 else 0 end) as second_class,
        SUM(CASE WHEN pclass = 3 then 1 else 0 end) as third_class
FROM titanic
GROUP BY survived

 ```

## Highest Salary In Department

Find the employee with the highest salary per department.

Output the department name, employee's first name along with the corresponding salary.

| Columna         | Tipo de dato |
|-----------------|--------------|
| address         | text         |
| age             | bigint       |
| bonus           | bigint       |
| city            | text         |
| department      | text         |
| email           | text         |
| employee_title  | text         |
| first_name      | text         |
| id              | bigint       |
| last_name       | text         |
| manager_id      | bigint       |
| salary          | bigint       |
| sex             | text         |
| target          | bigint       |



 ```sql
with cte as(
    SELECT first_name, department, salary, RANK() OVER(partition by department order by salary desc) as salary_rank
    FROM employee
)

SELECT department, first_name, salary
FROM cte
WHERE salary_rank = 1

 ```

## Highest Target Under Manager

Identify the employee(s) working under manager manager_id=13 who have achieved the highest target. Return each such employee’s first name alongside the target value. The goal is to display the maximum target among all employees under manager_id=13 and show which employee(s) reached that top value.

| Columna         | Tipo de dato |
|-----------------|--------------|
| address         | text         |
| age             | bigint       |
| bonus           | bigint       |
| city            | text         |
| department      | text         |
| email           | text         |
| employee_title  | text         |
| first_name      | text         |
| id              | bigint       |
| last_name       | text         |
| manager_id      | bigint       |
| salary          | bigint       |
| sex             | text         |
| target          | bigint       |





 ```sql
with cte as(
    SELECT first_name, target, RANK() OVER(order by target desc) as rango
    FROM salesforce_employees
    WHERE manager_id = 13
)


SELECT first_name, target
FROM cte
WHERE rango = 1


 ```

## Highest Cost Orders

Find the customer with the highest daily total order cost between 2019-02-01 to 2019-05-01. If customer had more than one order on a certain day, sum the order costs on daily basis. Output customer's first name, total cost of their items, and the date.

For simplicity, you can assume that every first name in the dataset is unique.

customers

| Columna        | Tipo de dato |
|----------------|--------------|
| address        | text         |
| city           | text         |
| first_name     | text         |
| id             | bigint       |
| last_name      | text         |
| phone_number   | text         |

orders

| Columna           | Tipo de dato |
|-------------------|--------------|
| cust_id           | bigint       |
| id                | bigint       |
| order_date        | date         |
| order_details     | text         |
| total_order_cost  | bigint       |



 ```sql
with cte as(
    SELECT c.first_name, o.order_date, SUM(o.total_order_cost) AS suma
    FROM customers c
    JOIN orders o on c.id = o.cust_id
    WHERE o.order_date BETWEEN '2019-02-01' AND '2019-05-01'
    GROUP BY o.order_date, c.first_name
)


SELECT *
FROM cte
WHERE suma in (select max(suma) as max from cte)


 ```

## Largest Olympics

Find the Olympics with the highest number of unique athletes. The Olympics game is a combination of the year and the season, and is found in the games column. Output the Olympics along with the corresponding number of athletes. The id column uniquely identifies an athlete.

| Columna  | Tipo de dato |
|----------|--------------|
| age      | double       |
| city     | text         |
| event    | text         |
| games    | text         |
| height   | double       |
| id       | bigint       |
| medal    | text         |
| name     | text         |
| noc      | text         |
| season   | text         |
| sex      | text         |
| sport    | text         |
| team     | text         |
| weight   | double       |
| year     | bigint       |



 ```sql
with cte as(
    SELECT games, count(distinct id) as conteo
    FROM olympics_athletes_events
    GROUP BY games
)

SELECT games, max(conteo)
FROM cte
group by games


 ```

## Top Businesses With Most Reviews


Find the top 5 businesses with most reviews. Assume that each row has a unique business_id such that the total reviews for each business is listed on each row. Output the business name along with the total number of reviews and order your results by the total reviews in descending order.


| Columna        | Tipo de dato |
|----------------|--------------|
| address        | text         |
| business_id    | text         |
| categories     | text         |
| city           | text         |
| is_open        | bigint       |
| latitude       | double       |
| longitude      | double       |
| name           | text         |
| neighborhood   | text         |
| postal_code    | text         |
| review_count   | bigint       |
| stars          | double       |
| state          | text         |


 ```sql
with cte as (
    SELECT *, RANK() OVER (ORDER BY review_count DESC) AS ranking
    FROM yelp_business
)

SELECT name, review_count
FROM cte
WHERE ranking <= 5
ORDER BY review_count DESC;


 ```

## Income By Title and Gender


Find the average total compensation based on employee titles and gender. Total compensation is calculated by adding both the salary and bonus of each employee. However, not every employee receives a bonus so disregard employees without bonuses in your calculation. Employee can receive more than one bonus.

Output the employee title, gender (i.e., sex), along with the average total compensation.

sf_employee

| Columna         | Tipo de dato |
|-----------------|--------------|
| address         | text         |
| age             | bigint       |
| city            | text         |
| department      | text         |
| email           | text         |
| employee_title  | text         |
| first_name      | text         |
| id              | bigint       |
| last_name       | text         |
| manager_id      | bigint       |
| salary          | bigint       |
| sex             | text         |
| target          | bigint       |


sf_bonus

| Columna         | Tipo de dato |
|-----------------|--------------|
| bonus           | bigint       |
| worker_ref_id   | bigint       |



 ```sql
with cte as (
    SELECT worker_ref_id, SUM(bonus) as bonus
    FROM sf_bonus
    GROUP BY worker_ref_id
)

SELECT s.employee_title, s.sex, AVG(salary + bonus)
FROM sf_employee s
JOIN cte on s.id = cte.worker_ref_id
GROUP BY s.employee_title, s.sex

 ```


## Matching Similar Hosts and Guests


Find matching hosts and guests pairs in a way that they are both of the same gender and nationality.

Output the host id and the guest id of matched pair.

airbnb_hosts

| Columna      | Tipo de dato |
|--------------|--------------|
| age          | bigint       |
| gender       | text         |
| host_id      | bigint       |
| nationality  | text         |

airbnb_guests

| Columna      | Tipo de dato |
|--------------|--------------|
| age          | bigint       |
| gender       | text         |
| guest_id     | bigint       |
| nationality  | text         |



 ```sql
with cte as (
    SELECT ah.host_id, ah.nationality, ah.gender, ah.age, ag.guest_id
    FROM airbnb_hosts ah
    JOIN airbnb_guests ag USING(nationality, gender)
)

SELECT distinct host_id, guest_id
FROM cte

 ```



## User with Most Approved Flags


Which user flagged the most distinct videos that ended up approved by YouTube? Output, in one column, their full name or names in case of a tie. In the user's full name, include a space between the first and the last name.

user_flags

| Columna        | Tipo de dato |
|----------------|--------------|
| flag_id        | text         |
| user_firstname | text         |
| user_lastname  | text         |
| video_id       | text         |


flag_review

| Columna         | Tipo de dato |
|-----------------|--------------|
| flag_id         | text         |
| reviewed_by_yt  | tinyint      |
| reviewed_date   | date         |
| reviewed_outcome| text         |




 ```sql
with union_table as(
    SELECT *, CONCAT(COALESCE (user_firstname, ''),' ', COALESCE (user_lastname, '')) as user_name
    FROM user_flags
    JOIN flag_review USING(flag_id)
),
ranked_users as (
    SELECT user_name, RANK() OVER (ORDER BY COUNT(DISTINCT video_id) DESC) AS ranking
    FROM union_table
    WHERE reviewed_outcome = 'APPROVED'
    GROUP BY user_name
)

SELECT *
FROM ranked_users
where ranking = 1


 ```

## Premium vs Freemium


Find the total number of downloads for paying and non-paying users by date. Include only records where non-paying customers have more downloads than paying customers. The output should be sorted by earliest date first and contain 3 columns date, non-paying downloads, paying downloads. Hint: In Oracle you should use "date" when referring to date column (reserved keyword).

ms_user_dimension

| Columna  | Tipo de dato |
|----------|--------------|
| acc_id   | bigint       |
| user_id  | bigint       |



ms_acc_dimension

| Columna          | Tipo de dato |
|------------------|--------------|
| acc_id           | bigint       |
| paying_customer  | text         |



ms_download_facts

| Columna    | Tipo de dato |
|------------|--------------|
| date       | date         |
| downloads  | bigint       |
| user_id    | bigint       |




 ```sql
SELECT 
  d.date,
  SUM(CASE WHEN a.paying_customer = 'yes' THEN d.downloads ELSE 0 END) AS premium,
  SUM(CASE WHEN a.paying_customer = 'no' THEN d.downloads ELSE 0 END) AS free
FROM ms_user_dimension u
JOIN ms_acc_dimension a ON u.acc_id = a.acc_id
JOIN ms_download_facts d ON u.user_id = d.user_id
GROUP BY d.date
having free > premium
ORDER BY d.date;


 ```

## Number of Streets Per Zip Code

Count the number of unique street names for each postal code in the business dataset. Use only the first word of the street name, case insensitive (e.g., "FOLSOM" and "Folsom" are the same). If the structure is reversed (e.g., "Pier 39" and "39 Pier"), count them as the same street. Output the results with postal codes, ordered by the number of streets (descending) and postal code (ascending).

| Columna               | Tipo de dato |
|-----------------------|--------------|
| business_address      | text         |
| business_city         | text         |
| business_id           | bigint       |
| business_latitude     | double       |
| business_location     | text         |
| business_longitude    | double       |
| business_name         | text         |
| business_phone_number | double       |
| business_postal_code  | double       |
| business_state        | text         |
| inspection_date       | date         |
| inspection_id         | text         |
| inspection_score      | double       |
| inspection_type       | text         |
| risk_category         | text         |
| violation_description | text         |
| violation_id          | text         |

 ```sql
WITH cte1 AS
  (SELECT CASE
              WHEN REGEXP_LIKE(business_address, "^[0-9]") = 1 THEN substring_index(substring_index(business_address, ' ', 2), ' ', -1)
              ELSE substring_index(business_address, ' ', 1)
          END AS street_name,
          business_postal_code AS postal_code
   FROM sf_restaurant_health_violations)
SELECT postal_code,
       count(DISTINCT street_name) AS num
FROM cte1
WHERE postal_code IS NOT NULL
GROUP BY postal_code
ORDER BY num DESC,
         postal_code ASC
 ```

## Election Results

The election is conducted in a city and everyone can vote for one or more candidates, or choose not to vote at all. Each person has 1 vote so if they vote for multiple candidates, their vote gets equally split across these candidates. For example, if a person votes for 2 candidates, these candidates receive an equivalent of 0.5 vote each. Some voters have chosen not to vote, which explains the blank entries in the dataset.

Find out who got the most votes and won the election. Output the name of the candidate or multiple names in case of a tie.

To avoid issues with a floating-point error you can round the number of votes received by a candidate to 3 decimal places.


| Columna  | Tipo de dato |
|----------|--------------|
| candidate| varchar      |
| voter    | varchar      |



 ```sql
with cte1 as (
    SELECT voter, COUNT(candidate) as n_votos
    FROM voting_results
    WHERE candidate <> ''
    GROUP BY voter
),
cte2 as (
    SELECT vr.voter, vr.candidate, ROUND(1 / cte1.n_votos, 3) as valor_voto
    FROM voting_results vr
    JOIN cte1 on cte1.voter = vr.voter
    WHERE candidate <> ''
),
cte3 as (
    SELECT candidate, sum(valor_voto) as valor_voto_total
    FROM cte2
    GROUP BY candidate
)

SELECT candidate
FROM cte3
ORDER BY valor_voto_total DESC
LIMIT 1


 ```

## Flags per Video

For each video, find how many unique users flagged it. A unique user can be identified using the combination of their first name and last name. Do not consider rows in which there is no flag ID.

| Columna        | Tipo de dato |
|----------------|--------------|
| flag_id        | text         |
| user_firstname | text         |
| user_lastname  | text         |
| video_id       | text         |



 ```sql
WITH cte as(
    SELECT *, CONCAT(COALESCE(user_firstname,''),  ' ', COALESCE(user_lastname, '')) as nombre_completo
    FROM user_flags
    WHERE flag_id is not null
)


SELECT video_id, count(DISTINCT nombre_completo)
FROM cte
GROUP BY video_id

 ```
