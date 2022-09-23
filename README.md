# Bank Customers Churn

<a href="https://app.powerbi.com/view?r=eyJrIjoiNDllMDBjMjQtYTJhMy00MDZiLWFiNGQtZTBkNGM4YzA4NDNhIiwidCI6IjM2OTk1Mzg5LWZjOWYtNGY2Ni04YjBlLTcyMWRmN2JiYWZiMCIsImMiOjl9&pageName=ReportSection2ca367e808080dfb2e0e">Dashboard in PowerBI</a>

***

This is a project from the Laboratoria & IBM Data Analyst Certification.

In this project, I had to choose by my own the topic and dataset that I would like to work with. My choice was a dataset about a bank's customers churn.

***

## What is Churn Rate?

The churn rate is the rate that shows the percentage of clients that stop doing business with the company in a time period. It's a very important metric to provide clarity on how well the business is retaining customers.

***

# Dataset

<a href="https://www.kaggle.com/datasets/santoshd3/bank-customers">Link here</a>

### Context

A dataset which contain some customers who are withdrawing their account from the bank due to some loss and other issues with the help this data we try to analyse and maintain accuracy.

### Variables

- RowNumber: Count the number of lines;
- CustomerId: ID from the client;
- Surname: Surname of relevant customer;
- CreditScore: It changes 350-850 in the dataset, express credit eligibility;
- Geography: Geography distribution;
- Gender: Customer gender;
- Age: Customer age;
- Tenure: How many years the customer works with this bank;
- Balance: Customer's total money in the account;
- NumOfProducts: Total of products that the customer uses;
- HasCrCard: Whether has credit card or not (customer has credit card is 1);
- IsActiveMember: Customer is active or not
- EstimatedSalary: Customers salary (yearly)
- Exited: Churn or not (Churn situation is 1)

It contais 1000 rows and 14 columns

***

# Import and Data Clean

I did all the exploratory analysis in Google Big Query.

After creating and importing the data, I ran the code in order to see if the dataset is complet.
```sql
SELECT
  *
FROM `proj6-bank-churn.data.data`;
```        
 ### Data Clean
 
 #### Checking for nulls
```sql
SELECT 
  *
FROM `proj6-bank-churn.data.data`
WHERE (1,2,3,4,5,6,7,8,9,10,11,12,13,14) IS NULL;
```
![image](https://user-images.githubusercontent.com/106877571/191948117-93a791c1-c91d-4c86-86fb-ad889ed88ce5.png)

It shows no results.

#### Drop columns that won't be used
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT 
  * EXCEPT (RowNumber, Surname)
FROM `proj6-bank-churn.data.data`;
```
#### Check if Gender is standardized
```sql
SELECT
  DISTINCT Gender
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191950386-7f47cc8e-c5a3-4d5a-b6d7-3a7cfbf265aa.png)

As it is, we won't need to work on it.

***

# Exploratory Analysis

#### Count total clients
```sql
SELECT
  COUNT(DISTINCT CustomerID) total_unique_clients
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191949042-aba91c5b-ca58-4c02-93bc-e4af0794ef77.png)

As our dataset has 1000 rows, we conclued that each row represents 1 client.

#### Identify all locations
```sql
SELECT
  DISTINCT Geography
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191949606-f9aee2ca-6e03-46c1-ba68-0a546a07260d.png)

#### Clients' Distribution by Geography
```sql
SELECT
  Geography,
  COUNT(DISTINCT CustomerId) Total,
  ROUND(((COUNT(DISTINCT CustomerId)/10000)*100),2) Percent
FROM `proj6-bank-churn.data.data`
GROUP BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191949931-eaffabc2-806e-46a0-8ab7-cac0e8c0a94b.png)

#### Clients' Distribution by Gender
```sql
SELECT
  Gender,
  COUNT(DISTINCT CustomerId) as Total,
  ROUND(((COUNT(DISTINCT CustomerId)/10000)*100),2) Percent
FROM `proj6-bank-churn.data.data`
GROUP BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191950757-4a7720a6-ea70-43f1-ba19-9d34e707f924.png)

As the dashboard will be showed and explained in Portuguese, I will translate the gender and add to the dataset.
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT
  *,
  CASE 
    WHEN Gender = 'Male' THEN 'Homem'
    ELSE 'Mulher'
  END AS Gender_2
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191951083-6e324170-0012-49d6-92d7-c0788f8672d4.png)

#### Check the max and min age to do a better segmentation
```sql
SELECT
  MIN(Age) min_age,
  MAX(Age) max_age
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191951383-190d7512-2bb4-40b1-aadf-3e0c08ba788f.png)

I've decided to segment in 3 categories: 18 to 40, 41 to 60 and over 60.
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT
  * EXCEPT(Age_Segment),
  CASE 
   WHEN Age BETWEEN 18 AND 40 THEN '1. 18 a 40'
   WHEN Age BETWEEN 41 AND 60 THEN '2. 41 a 60'
   ELSE '3. > 60'
  END AS Age_Segment
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191951758-61e17b66-0481-4976-89e1-2f01b18834ff.png)

##### Clients' Distribution by Age
```sql
SELECT
  Age_Segment,
  COUNT(DISTINCT CustomerId) as Total,
  ROUND(((COUNT(DISTINCT CustomerId)/10000)*100),2) Percent
FROM `proj6-bank-churn.data.data`
GROUP BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191951995-8d9a58b1-75f2-480a-b5fa-d3ba758f1849.png)

#### Explore tenure
```sql
SELECT
  MIN(Tenure) min_tenure,
  MAX(Tenure) max_tenure,
  AVG(Tenure) avg_tenure
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191952159-76275b1a-cbd3-4fb3-b364-367bcd139599.png)

As we can see, the minimum time of contract of our clients in this dataset is 0 year, which means that they still haven't completed 1 year of service, the maximum is 10 years, and the average time is 5 years of contract.

##### Clients' Distribution by Tenure 
```sql
SELECT
  Tenure,
  COUNT(DISTINCT CustomerId) as Total,
  ROUND(((COUNT(DISTINCT CustomerId)/10000)*100),2) Percent
FROM `proj6-bank-churn.data.data`
GROUP BY 1
ORDER BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191953648-893d0766-257b-4d0b-8f1e-48ce5f229465.png)

#### Active Members
```sql
SELECT
  COUNT(CustomerId) Active_Members
FROM `proj6-bank-churn.data.data`
WHERE IsActiveMember = 1;
```
![image](https://user-images.githubusercontent.com/106877571/191954046-a9f61598-c97f-4a46-a53f-7047c71d5395.png)

#### Inactive Members
```sql
SELECT
  COUNT(CustomerId) Inactive_Members
FROM `proj6-bank-churn.data.data`
WHERE IsActiveMember = 0;
```
![image](https://user-images.githubusercontent.com/106877571/191954201-3a81d62a-b9bc-4106-8898-9d5d5218294e.png)

#### Create a column that answers 'Yes' or 'No' for 'IsActiveMember'
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT 
  * EXCEPT(IsActiveMember_segment),
  CASE
    WHEN IsActiveMember = 1 THEN 'Sim'
    ELSE 'Não'
  END AS IsActiveMember_segment
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191954419-57c035d0-5b2c-4735-b6c4-edc97330491d.png)

#### Clients' Distribution by Number of Products
```sql
SELECT
  NumOfProducts,
  COUNT(CustomerId) Total_Clients,
  ROUND(((COUNT(CustomerId)/10000)*100),2) Percent
FROM `proj6-bank-churn.data.data`
GROUP BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191955432-70f1d198-60d4-4d99-a51a-2dcc59d8905f.png)

#### Clients' Distribution by Has Credit Cards or no
```sql
SELECT 
  HasCrCard,
  COUNT(CustomerId) Total_Clients,
  ROUND(((COUNT(CustomerId)/10000)*100),2) Percent
FROM `proj6-bank-churn.data.data`
GROUP BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191956062-a2d2fd6d-0b2f-4ee5-862e-bbf373049a25.png)

##### Create a column that answers 'Yes' or 'No' for 'HasCreditCard'
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT 
  *,
  CASE
   WHEN HasCrCard = 1 THEN 'Sim'
   ELSE 'Não'
  END AS HasCrCard_segment
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191956500-b1a5e51c-1b19-4eb2-8c55-0cb605027b30.png)

#### Check the max and min estimated_salary to do a better segmentation
```sql
SELECT
  MIN(EstimatedSalary) min_estimated_salary,
  MAX(EstimatedSalary) max_estimated_salary,
  AVG(EstimatedSalary) avg_estimated_salary
FROM `proj6-bank-churn.data.data`;
```

![image](https://user-images.githubusercontent.com/106877571/191956677-be2faa66-c05c-4356-bc69-261b44cecbce.png)

##### Estimated_salary segmentation
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT
  * EXCEPT(EstimatedSalary_segment),
  CASE
   WHEN EstimatedSalary < 10000 THEN '1. < 10.000'
   WHEN EstimatedSalary BETWEEN 10000 AND 50000 THEN '2. 10.000 a 50.000'
   WHEN EstimatedSalary BETWEEN 50001 AND 100000 THEN '3. 50.001 a 100.000'
   WHEN EstimatedSalary BETWEEN 100001 AND 150000 THEN '4. 100.001 a 150.000'
   ELSE '5. > 150.000'
  END AS EstimatedSalary_segment
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191956789-37a28db7-6f27-4cfa-8db1-6ef44b65c1c8.png)

#### Check the max and min balance to do a better segmentation
```sql
SELECT
  MIN(Balance) min_balance,
  MAX(Balance) max_balance,
  AVG(Balance) avg_balance
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191956890-2dcbab8b-b5b3-4f11-bdb7-0529c8f4ff4a.png)

##### Balance segmentation
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT
  * EXCEPT (Balance_Segment),
  CASE
   WHEN Balance < 50000 THEN '1. 0 a 50.000'
   WHEN Balance BETWEEN 50001 AND 100000 THEN '2. 50.001 a 100.000'
   WHEN Balance BETWEEN 100001 AND 150000 THEN '3. 100.001 a 150.000'
   WHEN Balance BETWEEN 150001 AND 200000 THEN '4. 150.001 a 200.000'
   ELSE '5. > 200.000'
  END AS Balance_Segment
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191957105-bcb28b88-ff89-449b-b7e9-b37e3c5889c3.png)

#### Check the max and min CreditScore to do a better segmentation
```sql
SELECT
  MAX(CreditScore) max_creditscore,
  MIN(CreditScore) min_creditscore,
  AVG(CreditScore) avg_creditscore
FROM `proj6-bank-churn.data.data`;
```
###### CreditScore segmentation considering the FICO Score
```sql
CREATE OR REPLACE TABLE `proj6-bank-churn.data.data`
AS
SELECT
  * EXCEPT(CreditScore_Segment),
  CASE
   WHEN CreditScore BETWEEN 300 AND 579 THEN '1. Fraco'
   WHEN CreditScore BETWEEN 580 AND 669 THEN '2. Justo'
   WHEN CreditScore BETWEEN 670 AND 739 THEN '3. Bom'
   WHEN CreditScore BETWEEN 740 AND 799 THEN '4. Muito Bom'
   ELSE '5. Excepcional'
 END AS CreditScore_Segment
FROM `proj6-bank-churn.data.data`;
```
![image](https://user-images.githubusercontent.com/106877571/191957402-4f292680-dc5f-428b-9bdb-d045ab3ce5b8.png)

##### Clients' Distribution by CreditScore Segmentation
```sql
SELECT
  CreditScore_Segment,
  COUNT(CustomerID) Total_Clients,
  ROUND(((COUNT(CustomerId)/10000)*100),2) percent_creditscore
FROM `proj6-bank-churn.data.data`
GROUP BY 1
ORDER BY 1;
```
![image](https://user-images.githubusercontent.com/106877571/191957718-8eb3c078-0be9-4176-bbaa-746be3353eef.png)

After understand our clients' profile, we will analyze the churn according to our variables, which is the main goal of this project.

***

# Churn Analysis

As I've already done the Exploratory Analysis and organized our data with all the necessary segmentations, the churn analysis will be done at Power BI. All the charts will be considered by percentage doing a comparative between churn and retention clients.

![image](https://user-images.githubusercontent.com/106877571/191993636-94e1a56a-90e5-4096-b8fe-da230e37331f.png)

We can see that the churn rate is 20,4% and the retention is 79,6%.

![image](https://user-images.githubusercontent.com/106877571/191994172-c2fe2532-d1cb-413d-8499-fc034517e194.png)

When we analyze the gender, we can see that despite the male customer is higher, the female churn is higher.

![image](https://user-images.githubusercontent.com/106877571/192061040-682e9605-0e7d-498d-a057-17c8cfe4b597.png)

Considering the age, we clearly see that the range between 46 and 65 has almost 50% of churn, which is incredible high. Even tough the clients majority is at range between 18 and 45, the churn rate is still significant for the range of 46 and 65.

![image](https://user-images.githubusercontent.com/106877571/192061323-e61cf4e0-6f9f-4f8f-b481-4d5a48ac3022.png)

Comparing when the client is and is not active, we can see that the inactive members has almost 2x churn than the active ones. 

![image](https://user-images.githubusercontent.com/106877571/192061409-7910424c-332f-49a9-a427-8ba10730880d.png)

Howerver, when it's about the credit card, it is irrelevant.

![image](https://user-images.githubusercontent.com/106877571/192061462-0d8e50b1-1c7a-4745-a138-447eefac9d16.png)

Such as the estimated salary, no relevance between it and churn.

![image](https://user-images.githubusercontent.com/106877571/192061498-e29a1ffe-c1c0-48bf-8beb-0007340ca63e.png)

But when we consider the balance, we clearly identify that the higher the balance, higher the churn.

![image](https://user-images.githubusercontent.com/106877571/192061551-bf010c75-9b9e-4797-820f-5c6e2d3fff56.png)

As I've done the Credit Score segmentation considering the FICO Score, we can see that it's also not relevant for our churn rate.

![image](https://user-images.githubusercontent.com/106877571/192061659-1a680c42-fa10-4ad9-9790-b3f52906a455.png)

Such as before, when we analyze the tenure, it's also not relevant for the churn rate. However, I would like to call atention for this less than one year metric.

![image](https://user-images.githubusercontent.com/106877571/192061742-1c9a3324-6c1d-46fb-9299-a0925e1a0fe7.png)

According to Accenture, during the first year the bank companies tend to have a 25% churn rate (and our client has 23%). However, the churn rate annual goal is 11% and our client is almost 2x more.

![image](https://user-images.githubusercontent.com/106877571/192061908-fe5339f3-bbbf-44f7-b464-1f04ef18ba79.png)

Considering the number of products that our clients have, we clearly see that 3 and 4 (mainly 4) has a expressive churn rate. And it's important to better understand the numbers.

![image](https://user-images.githubusercontent.com/106877571/192061996-f05690bb-d01c-4ad6-a497-33eef26ba078.png)

When we see in details the numbers from 4 products, we have less than 1% of clients, so we can't assume that it's relevant only because of the number of 4 products.

![image](https://user-images.githubusercontent.com/106877571/192062128-bef72d64-31a9-49cc-83cc-27ae981f0b68.png)

Now, looking for the churn between the countries, Germany has 2x more than France and Spain. And France has 2x more clients than Germany.

***

Conclusion and Recommendation

After all the data analysis, my conclusions and recommendations are:

- People aged between 46 and 65 have higher incidences of churn. As a recommendation, the bank could create new products and services specifically for this age group, offer lower interest rates for loans and improve private pension conditions, as this is an age group that is possibly looking for better profitability conditions for retirement;

- Germany is the country with the highest incidence of churn, with twice the percentage of France and Spain. As a recommendation, identify the differences between countries in terms of ATM availability and ease of access. And, through this analysis, incorporate changes in Germany;

- Inactive customers have almost double the percentage of active churn. With this, the recommendation is to segment these customers in order to identify better marketing strategies for engagement;

- As the account balance increases, the greater the incidence of churn. With this, it is possible to deduce that the products and services for this group of customers are not being satisfactory. Therefore, I recommend a satisfaction survey with these potential customers in order to identify better offer conditions in order to reduce churn;

- Women tend to have more churn. With this, it is recommended to attract the female audience through marketing campaigns such as advertising with women telling their stories of overcoming/financial planning and offering special rates;

- Although we have a high churn rate for customers with 3 or more products, it is only 3% of the total number of customers, which we cannot consider significant for our conclusion.

***

Next Step: Machine Learning Prediction Model.



