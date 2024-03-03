<h1 align='center'> Part A. Pizza Metrics </h1>  

**1.**  How many pizzas were ordered?
<details>
  <summary>Click to expand answer!</summary>

  ```sql
SELECT 
    COUNT(order_id) as pizzas
FROM
    customer_orders;
  ```
</details>

**Results:**

 pizzas|
----------------|
14|

**2.**  How many unique customer orders were made?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT
	COUNT(DISTINCT order_id) AS unique_orders
FROM
	customer_orders;
  ```
</details>

**Results:**

unique_orders|
-------------|
10|

**3.**  How many successful orders were delivered by each runner?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    R.runner_id,
    COUNT(O.order_id) AS Successful_orders
FROM 
    runners R
JOIN 
    runner_orders O USING(runner_id)
WHERE 
    O.pickup_time <> 'null'
GROUP BY 
    R.runner_id;
  ```
</details>

**Results:**

runner_id|successful_orders|
---------|-----------------|
1|                4|
2|                3|
3|                1|


**4.**  How many of each type of pizza was delivered?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
	
    P.pizza_name,
   COUNT( R.order_id) AS No_of_pizzas
FROM 
    customer_orders C
JOIN 
    pizza_names P On P.pizza_id = C.pizza_id
JOIN 
	runner_orders R On R.order_id = C.order_id 
WHERE 
    R.pickup_time <> 'null'
group by 
	P.pizza_name;
  ```
</details>

**Results:**

pizza_name|No_of_pizzas|
----------|--------------|
Meatlovers|             9|
Vegetarian|             3|

**5.**  How many Vegetarian and Meatlovers were ordered by each customer?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
Select C.Customer_id,
sum(Case
	When P.Pizza_name  = 'Meatlovers'Then 1 else 0
    end) as MeatLovers,
sum(Case
	When P.Pizza_name  = 'Vegetarian' Then 1 else 0
    end) as Vegetarian
From
	customer_orders C
JOIN
	pizza_names P using(pizza_id)
JOIN 
	runner_orders R On R.order_id = C.order_id 
WHERE 
    R.pickup_time <> 'null'
group by 
C.Customer_id;
  ```
</details>

**Results:**

customer_id|meat_lovers|vegetarian|
-----------|-----------|----------|
101|          2|         0|
102|          2|         1|
103|          3|         1|
104|          3|         0|
105|          0|         1|

**6.**  What was the maximum number of pizzas delivered in a single order?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql

Select Max_pizzas from(
Select C.order_id,count(C.order_id) as Max_pizzas
,
rank() over( order by count(C.order_id) desc) as Rnk
From
	customer_orders C
JOIN
	pizza_names P using(pizza_id)
JOIN 
	runner_orders R On R.order_id = C.order_id 
WHERE 
    R.pickup_time != 'null'
Group by C.order_id) X
where Rnk = 1;
  ```
</details>

**Results:**

max_delivered_pizzas|
--------------------|
3|

**7.**  For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    C.Customer_id,
    SUM(CASE
        WHEN
            (c.exclusions <> 'null'
                AND c.exclusions IS NOT NULL
                AND c.exclusions <> '')
                OR (c.extras <> 'null'
                AND c.extras IS NOT NULL
                AND c.exclusions <> '')
        THEN 1
        ELSE 0 END) AS Changes_made,
    SUM(CASE
        WHEN
            (c.exclusions <> 'null'
                AND c.exclusions IS NOT NULL
                AND c.exclusions <> '')
                OR (c.extras <> 'null'
                AND c.extras IS NOT NULL
                AND c.exclusions <> '')
        THEN 0
        ELSE 1 END) AS NO_changes_made
FROM
    customer_orders C
        JOIN
    pizza_names P USING (pizza_id)
        JOIN
    runner_orders R ON R.order_id = C.order_id
WHERE
    R.pickup_time <> 'null'
GROUP BY C.Customer_id;
  ```
</details>

**Results:**

customer_id|Changes_made|NO_changes_made|
-----------|------------|----------|
101|           0|         2|
102|           0|         3|
103|           3|         0|
104|           2|         1|
105|           1|         0|

**8.**  How many pizzas were delivered that had both exclusions and extras?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    COUNT(CASE
        WHEN
            (C.exclusions IS NOT NULL
                AND C.exclusions <> 'null'
                AND C.exclusions <> '')
                AND (C.extras IS NOT NULL
                AND C.extras <> 'null'
                AND C.extras <> '')
        THEN
            C.order_id
        ELSE NULL
    END) AS Pizzas_with_both
FROM
    customer_orders C
        JOIN
    pizza_names P USING (pizza_id)
        JOIN
    runner_orders R ON R.order_id = C.order_id
WHERE
    R.pickup_time <> 'null';
  ```
</details>

**Results:**

Pizzas_with_both|
----------------|
1|

**9.**  What was the total volume of pizzas ordered for each hour of the day?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    EXTRACT(HOUR FROM order_time) AS hour_of_day_24h, COUNT(order_id) AS pizzas_ordered
FROM
    customer_orders
GROUP BY hour
ORDER BY hour;
  ```
</details>

**Results:**

hour_of_day_24h|pizzas_ordered|
---------------|---------------|
11             |              1|
13             |             3|
18             |            3|
19             |            1|
21             |            3|
23             |             3| 

**10.**  What was the volume of orders for each day of the week?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    DAYNAME(order_time) AS day_of_week, 
    COUNT(order_id) AS Pizzas
FROM
    customer_orders
GROUP BY 
	day_of_week 
    , DAYOFWEEK(order_time)
ORDER BY 
	DAYOFWEEK(order_time);
  ```
</details>

**Results:**

day_of_week|Pizzas|
-----------|----------------|
Wednesday     |               5|
Thursday     |               3|
Friday     |               1|
Saturday   |               5|





<h1 align='center'> Part B. Runner and Customer Experience </h1>
  
**1.**  How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    DATE(registration_date - INTERVAL WEEKDAY(registration_date) DAY) + INTERVAL 4 DAY AS starting_week,
    COUNT(runner_id) AS runners
FROM
    runners
GROUP BY week;
  ```
</details>

**Results:**

starting_week|runners|
-------------|-----------------|
2021-01-01|                2|
2021-01-08|                1|
2021-01-15|                1|

**2.**  What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT
    R.runner_id,
    AVG(MINUTE(R.pickup_time)) AS avg_time_min
FROM
    runner_orders R
JOIN
    customer_orders C ON R.order_id = C.order_id
WHERE
    R.pickup_time IS NOT NULL
GROUP BY
    R.runner_id;
  ```
</details>

**Results:**

|Runner_id|avg_pickup_time|
|------------|-
|1|24.833|
|2|40.80
3|10.00|

**3.**  Is there any relationship between the number of pizzas and how long the order takes to prepare?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
With CTE as (select 
	count(pizza_id) as NO_of_pizzas,
	TIMESTAMPDIFF(MINUTE, C.order_time, R.pickup_time) as TD
FROM
    runner_orders R
JOIN
    customer_orders C ON R.order_id = C.order_id
WHERE
    R.pickup_time IS NOT NULL and timediff( R.pickup_time,C.order_time) is not null
GROUP BY
    R.order_id,TD
)
SELECT 
    No_of_pizzas, AVG(TD) AS AVG_time
FROM
    CTE
GROUP BY NO_of_pizzas;
  ```
</details>

**Results:**

|No_of_pizzas|AVG_time|
----------|----------|
1	|12.0000
2	|18.0000
3	|29.0000

**4.**  What was the average distance traveled for each customer?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    C.customer_id,
    ROUND(AVG(CAST(REPLACE(distance, 'km', '') AS DECIMAL (3 , 1 ))),
            2) AS avg_distance
FROM
    runner_orders R
        JOIN
    customer_orders C ON R.order_id = C.order_id
WHERE
    R.distance <> 'null'
GROUP BY C.customer_id;
  ```
</details>

**Results:**

customer_id|avg_distance|
-----------|------------|
101|       20.00|
102|       18.40|
103|       23.40|
104|       10.00|
105|       25.00|

**5.**  What was the difference between the longest and shortest delivery times for all orders?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    MAX(CAST(REGEXP_REPLACE(duration, '[^0-9]', '') AS UNSIGNED))
    - MIN(CAST(REGEXP_REPLACE(duration, '[^0-9]', '') AS UNSIGNED)) AS Duration
FROM
    runner_orders
WHERE
    duration <> 'null';
  ```
</details>

**Results:**

Duration|
--------|
30|

**6.**  What was the average speed for each runner for each delivery and do you notice any trend for these values?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    R.runner_id,
   R.order_id,
        ROUND(
            AVG(CAST(REPLACE(R.distance, 'km', '') AS DECIMAL(3, 1)))/
        AVG(CAST(REGEXP_REPLACE(R.duration, '[^0-9]', '') AS UNSIGNED)), 2) AS Duration_U
FROM
    runner_orders R
JOIN
    customer_orders C ON R.order_id = C.order_id
WHERE
    R.distance <> 'null'
GROUP BY 
    R.runner_id, 
    R.order_id
ORDER BY
	 R.runner_id, 
    R.order_id;
  ```
</details>

**Results:**

|runner_id|order_id|Duration|
|----------|-------|----------|
|1|	1|	0.63
|1	|2	|0.74
|1	|3	|0.67
|1	|10	|1.00
|2	|4	|0.59
|2	|7	|1.00
|2	|8	|1.56
|3	|5	|0.67

**7.**  What is the successful delivery percentage for each runner?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    runner_id,
    ROUND(SUM(CASE
                WHEN pickup_time = 'null' THEN 0
                ELSE 1
            END) / COUNT(order_id),
            2) * 100 AS successful_delivery_percentage
FROM
    runner_orders
GROUP BY runner_id;
  ```
</details>

**Results:**

runner_id | successful_delivery_percentage
---------------|-----|
1	|100.00
2	|75.00
3	|50.00



<h1 align='center'> Part C. Ingredient Optimization </h1> 

**1.**  What are the standard ingredients for each pizza?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    PN.pizza_name,
    GROUP_CONCAT(PT.topping_name
        SEPARATOR ',') AS std_ingredients
FROM
    pizza_recipes AS PR
        JOIN
    pizza_toppings PT ON FIND_IN_SET(PT.topping_id,
            REPLACE(PR.toppings, ' ', '')) > 0
        JOIN
    pizza_names AS PN ON PR.pizza_id = PN.pizza_id
GROUP BY PN.pizza_name;
  ```
</details>

**Results:**
pizza_name| std_ingredients
----------|----------
Meatlovers	|Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami
Vegetarian	|Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce

**2.**  What was the most commonly added extra?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
WITH cte AS
     (SELECT substring_index(extras,',', 1) AS extras1 
      FROM customer_orders
     )
SELECT COUNT(topping_name) as commonly_added_extra , topping_name FROM cte JOIN pizza_toppings p ON p.topping_id = cte.extras1
GROUP BY topping_name
LIMIT 1;
  ```
</details>

**Results:**

commonly_added_extra|topping_name|
-------------------|------------
4	|Bacon   |


**3.**  What was the most common exclusion?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
WITH cte AS
(SELECT substring_index(exclusions,',', 1) AS exclusions
  FROM customer_orders
  ) 
SELECT COUNT(topping_name) as common_exclusion , topping_name FROM cte JOIN pizza_toppings p ON p.topping_id = cte.exclusions
GROUP BY topping_name 
LIMIT 1;
  ```
</details>

**Results:**

common_exclusion|topping_name|
-------------------|------------
4	|Cheese  |

## Unanswered
4) Generate an order item for each record in the customers_orders table in the format of one of the following:
Meat Lovers
Meat Lovers - Exclude Beef
Meat Lovers - Extra Bacon
Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

5) Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

6) What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?



<h1 align='center'>Part D. Pricing & Ratings</h1>

**1.**  If a Meat Lovers pizza costs \$12 and Vegetarian costs \$10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
 SUM(CASE WHEN pizza_id=1 THEN 12
    WHEN pizza_id = 2 THEN 10
    END) AS Total_earnings
 FROM runner_orders r
JOIN customer_orders c ON c.order_id = r.order_id
WHERE r.pickup_time <>  'null';
  ```
</details>

**Results:**

Total_earnings|
---------------------------------|
138|

**2.**  What if there was an additional $1 charge for any pizza extras?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
WITH cte AS
(SELECT 
 (CASE WHEN pizza_id=1 THEN 12
    WHEN pizza_id = 2 THEN 10
    END) AS pizza_cost, 
    c.exclusions,
    c.extras
 FROM runner_orders r
JOIN customer_orders c ON c.order_id = r.order_id
WHERE r.pickup_time <>  'null')
SELECT 
 SUM(CASE WHEN extras IS NULL or extras ='' or extras='null' THEN pizza_cost
  WHEN LENGTH(extras) = 1 THEN pizza_cost + 1
        ELSE pizza_cost + 2
        END ) as Extra_price
FROM cte;
  ```
</details>

**Results:**

Extra_price|
------------|
142|

**3.**  The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
DROP TABLE IF EXISTS TABLE ratings;
CREATE TABLE ratings 
 (order_id INTEGER,
    rating INTEGER);
INSERT INTO ratings
 (order_id ,rating)
VALUES 
(1,3),
(2,4),
(3,5),
(4,2),
(5,1),
(6,3),
(7,4),
(8,1),
(9,3),
(10,5);
  ```
</details>

**Results:**

order_id|rating|
--------|------|
1	|3
2	|4
3	|5
4	|2
5	|1
6	|3
7	|4
8	|1
9	|3
10|5

**4.**  Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
* customer_id
* order_id
* runner_id
* rating
* order_time
* pickup_time
* Time between order and pickup
* Delivery duration
* Average speed
* Total number of pizzas

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    C.customer_id,
    COUNT(pizza_id) AS Total_number_of_pizzas,
    C.order_id,
    R.runner_id,
    RT.rating,
    C.order_time,
    R.pickup_time,
    AVG(CAST(REGEXP_REPLACE(R.duration, '[^0-9]', '') AS UNSIGNED)) AS Duriation,
    ROUND(AVG(CAST(REPLACE(distance, 'km', '') AS DECIMAL (3 , 1 ))),
            2) AS Avg_distance,
    ROUND(AVG(CAST(REPLACE(R.distance, 'km', '') AS DECIMAL (3 , 1 ))) / AVG(CAST(REGEXP_REPLACE(R.duration, '[^0-9]', '') AS UNSIGNED)),
            2) AS AVG_speed
FROM
    customer_orders C
        JOIN
    runner_orders R ON C.order_id = R.order_id
        JOIN
    ratings RT ON R.order_id = RT.order_id
WHERE
    pickup_time <> 'null'
GROUP BY C.customer_id , C.order_id , R.runner_id , RT.rating , C.order_time , R.pickup_time
ORDER BY C.customer_id;
  ```
</details>

**Results:**
| customer_id | Total_number_of_pizzas | order_id | runner_id | rating | order_time           | pickup_time          | Duration | Avg_distance | AVG_speed |
|-------------|------------------------|----------|-----------|--------|----------------------|----------------------|----------|--------------|-----------|
| 101         | 1                      | 1        | 1         | 3      | 2020-01-01 18:05:02  | 2020-01-01 18:15:34  | 32.0000  | 20.00        | 0.63      |
| 101         | 1                      | 2        | 1         | 4      | 2020-01-01 19:00:52  | 2020-01-01 19:10:54  | 27.0000  | 20.00        | 0.74      |
| 102         | 2                      | 3        | 1         | 5      | 2020-01-02 23:51:23  | 2020-01-03 00:12:37  | 20.0000  | 13.40        | 0.67      |
| 102         | 1                      | 8        | 2         | 1      | 2020-01-09 23:54:33  | 2020-01-10 00:15:02  | 15.0000  | 23.40        | 1.56      |
| 103         | 3                      | 4        | 2         | 2      | 2020-01-04 13:23:46  | 2020-01-04 13:53:03  | 40.0000  | 23.40        | 0.59      |
| 104         | 1                      | 5        | 3         | 1      | 2020-01-08 21:00:29  | 2020-01-08 21:10:57  | 15.0000  | 10.00        | 0.67      |
| 104         | 2                      | 10       | 1         | 5      | 2020-01-11 18:34:49  | 2020-01-11 18:50:20  | 10.0000  | 10.00        | 1.00      |
| 105         | 1                      | 7        | 2         | 4      | 2020-01-08 21:20:29  | 2020-01-08 21:30:45  | 25.0000  | 25.00        | 1.00      |


**5.**  If a Meat Lovers pizza was \$12 and Vegetarian \$10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometer traveled - how much money does Pizza Runner have left over after these deliveries?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
SELECT 
    runner_id,
    SUM(CASE
        WHEN pizza_id = 1 THEN 12
        WHEN pizza_id = 2 THEN 10
    END) - SUM((r.distance + 0) * 0.3) AS pizza_cost,
    SUM(CASE
        WHEN pizza_id = 1 THEN 12
        WHEN pizza_id = 2 THEN 10
    END) AS pizza_only,
    (SUM(r.distance + 0) * 0.3) AS distance_cost
FROM
    runner_orders r
JOIN 
	customer_orders c ON c.order_id = r.order_id
WHERE
    pickup_time <> 'null'
GROUP BY runner_id
  ```
</details>

**Results:**

| runner_id | pizza_cost | pizza_only | distance_cost |
|-----------|------------|------------|---------------|
| 1         | 43.96      | 70         | 26.04         |
| 2         | 20.42      | 56         | 35.58         |
| 3         | 9.00       | 12         | 3.00          |


<h1 align='center'>Part E. Bonus Question</h1>

**1.**  If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?

<details>
  <summary>Click to expand answer!</summary>

  ##### Answer
  ```sql
DROP TABLE IF EXISTS temp_pizza_names;
CREATE TEMPORARY TABLE temp_pizza_names AS (
	SELECT *
  	FROM
  		pizza_runner.pizza_names
);

INSERT INTO temp_pizza_names
VALUES
  (3, 'Supreme');


DROP TABLE IF EXISTS temp_pizza_recipes;
CREATE TABLE temp_pizza_recipes AS (
	SELECT *
  	FROM
  		pizza_runner.pizza_recipes
);

INSERT INTO temp_pizza_recipes
VALUES
  (3, '1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12');
 
SELECT
	t1.pizza_id,
	t1.pizza_name,
	t2.toppings
FROM 
	temp_pizza_names AS t1
JOIN
	temp_pizza_recipes AS t2
ON
	t1.pizza_id = t2.pizza_id;
  ```
</details>

**Results:**

pizza_id|pizza_name|toppings                             |
--------|----------|-------------------------------------|
1|Meatlovers|1, 2, 3, 4, 5, 6, 8, 10              |
2|Vegetarian|4, 6, 7, 9, 11, 12                   |
3|Supreme   |1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12|

