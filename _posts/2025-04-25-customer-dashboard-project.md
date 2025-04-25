---
title: "Data Analyst: Dealership Customer Performance Dashboard Project (WIP)"
date: 2025-03-03  00:00:00 +1000
categories: [Data Analyst]
tags: [Data Analyst]
author: lkarslake
description: "A Data Analysis project"
image:
  path: /assets/img/Dashboard.png
math: true
---
---
**NOTE**

The repository containing all of the project files can be found here: [Project files](https://github.com/lkarslake/customer-dashboard-project)

---


In this project we will simulate a customer service performance dashboard for a set of branded dealerships in Victoria.
We will generate synthesized data using Python for this simulation.

## Background
We are tasked to derive insights into dealership performance in relation to customer service over a fiscal year. The company values customer satisfaction, service and life time value. We can measure these performance metrics with a wide array of KPIs. In our case, I believe the following are the best choice: 

### KPIs
- Customer Satisfaction Score (CSAT), which will allow us to measure customer satisfaction with a 5 point scale on a employee level. This is important for gauging employee customer service. This is measured by: $$\text{Total number of 4-5 ratings}\;/\;\text{Total number of ratings}$$
- Average customer lifespan, which will allow us to measure the life time value to see if our efforts are focused on building customer relations. This is measured by:
$$\text{Last purchase date upto fiscal year} \; - \; \text{First purchase date}$$
- Sentiment Score, which will allow us to measure the general social media sentiment on a dealership level. This is important to see if the dealerships social media, one of the biggest forms of natural advertisement, is poorly regarded. This is measured by:
$$AVG(\text{Sentiment Score})$$

Due to the fact the data is not from a real source, the insights generated will not be reflective of the real world. Instead we will focus on the process rather than the results.

### Mockup
![image info](/assets/img/mockup.png)


With this information we can gauge the data we may need.

### Required Data
- Transaction Data: To see per transaction customer satisfaction
	- Transaction ID
	- Date
	- Dealer ID
	- Dealership ID
	- Customer ID
	- Car ID
- Dealership Data: To visualize the dealerships on a map and access the total marketing costs
	- Dealership ID
	- Postcode
	- State
	- Total Marketing Cost
- 

## Synthesizing the data
For this task we will use Python including the Faker package. These files can be found in the GitHub repository.

## Data Modelling
Now that we have our data, we need to connect it and model it in Tableau. Our data is in the form of CSV files so we can drop it straight in. We identify the following as fact tables: transactions, feedback, and the rest as dimension tables: cars, customers, employees, dealerships. We connect the dimension tables to the fact tables, inspect the data and proceed to the data visualization.

## Data Visualization
As show in the mockup, we need five visuals- CSAT KPI card, CLV:CAC KPI card, Sentiment Score card, CSAT per Employee bar chart, CSAT per Dealership map. 

### KPI cards
For the KPI cards we need to display the main FY25 metric: FY25 CSAT, FY25 CLV:CAC, and FY25 Sentiment Score respectively, as well as a % difference from the previous fiscal year. This % difference is important to illustrate whether or not the result is good or requires attention. So we define the following calculated fields:

#### CSAT KPI Measures

```
[FY25 CustomerSatisfaction] 

=IF (YEAR([Date Of Sale]) = 2024 AND MONTH([Date Of Sale]) >= 7) OR (YEAR([Date Of Sale]) = 2025 AND MONTH([Date Of Sale]) <= 6) THEN [CustomerSatisfaction]
END
```
(Same for FY24)

```
[FY25 CSAT]

=SUM(IF [FY25 Customer Satisfaction] >= 4 THEN 1 ELSE 0 END) / COUNT([FY25 Customer Satisfaction])
```
(Same for FY24)

```
[% CSAT Diff FY24-25 - POS]

IF (([FY25 CSAT] - [FY24 CSAT]) / [FY24 CSAT]) >= 0 THEN ([FY25 CSAT] - [FY24 CSAT]) / [FY24 CSAT] END
```

```
[% CSAT Diff FY24-25 - NEG]

IF (([FY25 CSAT] - [FY24 CSAT]) / [FY24 CSAT]) < 0 THEN ([FY25 CSAT] - [FY24 CSAT]) / [FY24 CSAT] END
```

Separation of the fields will allow us to do some conditional color formatting.


CLV:CAC KPI Measures

Sentiment Score KPI Measure




For the visual, we will use the text mark.

The final result is:
![image info](/assets/img/Dashboard.png)
