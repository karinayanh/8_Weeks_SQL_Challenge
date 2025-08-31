#  **Case Study #2: Pizza Runner**

<img width="352" height="344" alt="image" src="https://github.com/user-attachments/assets/11d6aa29-62ca-4e70-8422-9e9dd95cc4b7" />

The entire case can be viewed [here](https://8weeksqlchallenge.com/case-study-2/)

## **Introduction**
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

## **Available Data**

Danny has prepared for us an entity relationship diagram of his database design but requires further assistance to clean his data and apply some basic calculations so he can better direct his runners and optimise Pizza Runner’s operations.

The tables available are: 

- Runners: runner_id, registration_date
- Customer_orders: order_id, customer_id, pizza_id, exclusions, extras, order_time
- Runner_orders: order_id, runner_id, pickup_time, distance, duration, cancellation
- Pizza_names: pizza_id , pizza_name
- Pizza_recipes: pizza_id, toppings 
- Pizza_toppings: topping_id, topping_name

<img width="556" height="262" alt="image" src="https://github.com/user-attachments/assets/efa884ff-a699-466b-b100-5d670fff4f0d" />

## **Case Study Questions**

### **A - Pizza Metrics**

**1. How many pizzas were ordered?**

2. How many unique customer orders were made?
3. How many successful orders were delivered by each runner?
4. How many of each type of pizza was delivered?
5. How many Vegetarian and Meatlovers were ordered by each customer?
6. What was the maximum number of pizzas delivered in a single order?
7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
8. How many pizzas were delivered that had both exclusions and extras?
9. What was the total volume of pizzas ordered for each hour of the day?
10. What was the volume of orders for each day of the week?
