### datawithdanny
#### EMAIL MARKETING CASE STUDY OVERVIEW

We have been tasked by the DVD Rental Co marketing team to help them generate the analytical inputs required to drive their very first customer email campaign.
The marketing team expects their personalised emails to drive increased sales and engagement from the DVD Rental Co customer base.
The main initiative is to share insights about each customer’s viewing behaviour to demonstrate DVD Rental Co’s relentless focus on customer experience.
The insights requested by the marketing team include key statistics about each customer’s top 2 categories and favourite actor. There are also 3 personalised recommendations based off each customer’s previous viewing history as well as titles which are popular with other customers.
<p align="center">
    <img src="https://imgur.com/a/b8wkZtQ">

<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">
</p>

[![forthebadge](./../images/badges/version-1.0.svg)]()

##### A. Category Insights
###### 1. Top Category
What was the top category watched by total rental count?
How many total films have they watched in their top category and how does it compare to the DVD Rental Co customer base?
How many more films has the customer watched compared to the average DVD Rental Co customer?
How does the customer rank in terms of the top X% compared to all other customers in this film category?
What are the top 3 film recommendations in the top category ranked by total customer rental count which the customer has not seen before?

###### 2. Second Category
What is the second ranking category by total rental count?
What proportion of each customer’s total films watched does this count make?
What are top 3 recommendations for the second category which the customer has not yet seen before?

##### B. Actor Insights
Which actor has featured in the customer’s rental history the most?
How many films featuring this actor has been watched by the customer?
What are the top 3 recommendations featuring this same actor which have not been watched by the customer?

------
### SOLUTION VISUALIZATION
## TOP CATEGORIES AND TOP CATEGORY INSIGHTS
#### 1. Create temp table to join relevant table
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

#### 2. Calculate count of each customer and category. Create temp table category_counts
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

#### 3. Calculate total number of films customer has watched per each top 2 categories, [rental_count]. Create temp table top 2_category
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

#### 4. Calculate how many more films has the customer watched compared to the average DVD Rental Co customer
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

#### 5. Calculate how does the customer rank in terms of the top X% compared to all other customers in this film category
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">
  
#### 6. Calculate the total films that customer has watched in their history rental
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">
  
#### 7. Generate 1st category insight - create temp table top_1_category_insight
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

#### 8. Generate 2nd category insight - create temp table top_2_category_insight
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

## CATEGORY RECOMMENDATIONS
#### 1. Generate a summarised film count table with the category included. Create temp table film_count
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">
  
#### 2. Create a previously watched films for the top 2 categories to exclude for each customer
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">
  
#### 3. Create temp table final for category_recommendation
<p align="center">
    <img src="./../images/sql-masterclas-banner.png" alt="sql-masterclass-banner">

## TOP ACTOR & ACTOR INSIGHTS
#### 1. Create join temp table for actor dataset
#### 2. Create temp table for actor_rental_count - Calculate the actor that customers watches
#### 3. Create temp table for actor_rental_count - Calculate the actor that customers watches THE MOSTTT

## ACTOR RECOMMENDATIONS
#### 1. Create table count for all actor and all film 
#### 2. Create exclusion table for customer, title, actor
#### 3. Create table join for top 1 ranking actor with title, set ranking partition by customer and actor. And create table anti join with exclusion table to find the final result table
