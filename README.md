# Project Approach
I got this dataset from Kaggle, and it contains information such as customers' personal details, satisfaction scores, preferred payment mode, days since the last order, and cashback amount. I used SQL (Azure Data Studio) to clean and analyze this dataset, and performed visualizations using Microsoft Power BI. This analysis is divided into several stages: data cleaning, data exploration, an insight section, and recommendations.

# Data Cleaning
Before embarking on any analysis, it is essential to ensure the dataset is clean and reliable. The data cleaning process involves handling missing values, correcting inconsistencies, and formatting the data for analysis. In this project, we carefully cleaned the dataset to ensure the accuracy and integrity of our findings.  

**1. Finding the total number of customers**  

SELECT DISTINCT COUNT(CustomerID) as TotalNumberOfCustomers  
FROM ecommercechurn  

There are 5,630 customers found in this dataset  

**2. Checking for duplicate rows**  

SELECT CustomerID, COUNT (CustomerID) as Count  
FROM ecommercechurn  
GROUP BY CustomerID  
Having COUNT (CustomerID) > 1  

The query returns an empty table, showing there are no duplicate rows.  

**3. Checking for null values**  

We start out by finding the null count of columns with null values present.  

SELECT 'Tenure' as ColumnName, COUNT(*) AS NullCount  
FROM ecommercechurn  
WHERE Tenure IS NULL  
UNION  
SELECT 'WarehouseToHome' as ColumnName, COUNT(*) AS NullCount  
FROM ecommercechurn  
WHERE warehousetohome IS NULL  
UNION  
SELECT 'HourSpendonApp' as ColumnName, COUNT(*) AS NullCount  
FROM ecommercechurn  
WHERE hourspendonapp IS NULL
UNION  
SELECT 'OrderAmountHikeFromLastYear' as ColumnName, COUNT(*) AS NullCount 
FROM ecommercechurn  
WHERE orderamounthikefromlastyear IS NULL 
UNION  
SELECT 'CouponUsed' as ColumnName, COUNT(*) AS NullCount   
FROM ecommercechurn  
WHERE couponused IS NULL 
UNION
SELECT 'OrderCount' as ColumnName, COUNT(*) AS NullCount 
FROM ecommercechurn
WHERE ordercount IS NULL 
UNION
SELECT 'DaySinceLastOrder' as ColumnName, COUNT(*) AS NullCount 
FROM ecommercechurn
WHERE daysincelastorder IS NULL 

CouponUsed, DaysSinceLastOrder, HourSpendOnApp, OrderAmountHikeFromLastYear, OrderCount, Tenure, and WarehouseToHome all have null values present, and the number of null values present for each column can be seen in the above table.

**3.1 Handling null values**

In order to handle these null values, we will fill the nulls with the mean value of their respective columns.

UPDATE ecommercechurn
SET Hourspendonapp = (SELECT AVG(Hourspendonapp) FROM ecommercechurn)
WHERE Hourspendonapp IS NULL 

UPDATE ecommercechurn
SET tenure = (SELECT AVG(tenure) FROM ecommercechurn)
WHERE tenure IS NULL 

UPDATE ecommercechurn
SET orderamounthikefromlastyear = (SELECT AVG(orderamounthikefromlastyear) FROM ecommercechurn)
WHERE orderamounthikefromlastyear IS NULL 

UPDATE ecommercechurn
SET WarehouseToHome = (SELECT  AVG(WarehouseToHome) FROM ecommercechurn)
WHERE WarehouseToHome IS NULL 

UPDATE ecommercechurn
SET couponused = (SELECT AVG(couponused) FROM ecommercechurn)
WHERE couponused IS NULL 

UPDATE ecommercechurn
SET ordercount = (SELECT AVG(ordercount) FROM ecommercechurn)
WHERE ordercount IS NULL 

UPDATE ecommercechurn
SET daysincelastorder = (SELECT AVG(daysincelastorder) FROM ecommercechurn)
WHERE daysincelastorder IS NULL 

Now that the null values have been replaced with their mean, there are no more null values left.

**4. Creating a new column from an already existing “churn” column**

While going through the dataset, we noticed that the churn column contained 0 and 1 values. 0 means that a customer did not churn, while 1 means that a customer churned. It is difficult to remember which is for what, so we will create a new column called ‘CustomerStatus’ that shows “Stayed” when a customer did not churn and “Churned” when a customer churned.

ALTER TABLE ecommercechurn
ADD CustomerStatus NVARCHAR(50)

UPDATE ecommercechurn
SET CustomerStatus = 
CASE 
    WHEN Churn = 1 THEN 'Churned' 
    WHEN Churn = 0 THEN 'Stayed'
END 

The new column “CustomerStatus” has two distinct values called "churned" and "stayed," as earlier described.

**5. Creating a new column from an already existing “complain” column**

Very similar to what we did in the section above, we noticed that the complain column also contained 0 and 1. ‘0’ means that the customer did not record any complaints, while ‘1’ means the customer recorded a complaint. For clarity purposes, we will create a new column called ‘ComplainRecieved’ that shows “No” when a customer did not complain and “Yes” when a customer complained.

ALTER TABLE ecommercechurn
ADD ComplainRecieved NVARCHAR(10)

UPDATE ecommercechurn
SET ComplainRecieved =  
CASE 
    WHEN complain = 1 THEN 'Yes'
    WHEN complain = 0 THEN 'No'
END

The new column “complainrecieved” has two distinct values called “yes” and “no,” as earlier described.

**6. Checking values in each column for correctness and accuracy**

After going through each column, we noticed some redundant values in some columns and a wrongly entered value. This will be explored and fixed.

**6.1 Fixing redundancy in “PreferedLoginDevice” Column**

select distinct preferredlogindevice 
from ecommercechurn

Notice that phone and mobile phone appear in the column, but they mean the same thing. So we will replace the mobile phone with phone.

UPDATE ecommercechurn
SET preferredlogindevice = 'phone'
WHERE preferredlogindevice = 'mobile phone'

The result above shows the redundancy has been fixed.

**6.2 Fixing redundancy in “PreferedOrderCat” Column**

select distinct preferedordercat 
from ecommercechurn

Notice that mobile phone and phone appear in the column, but they mean the same thing. So we will replace the phone with mobile phone as the category name.

UPDATE ecommercechurn
SET preferedordercat = 'Mobile Phone'
WHERE Preferedordercat = 'Mobile'

The result above shows the redundancy has been fixed.

**6.3 Fixing redundancy in “PreferredPaymentMode” Column**

select distinct PreferredPaymentMode 
from ecommercechurn

Notice that Cash on Delivery and COD both appear in the column, but they mean the same thing. So we will replace COD with Cash on Delivery.

UPDATE ecommercechurn
SET PreferredPaymentMode  = 'Cash on Delivery'
WHERE PreferredPaymentMode  = 'COD'

The result above shows the redundancy has been fixed.

**6.4 Fixing wrongly entered values in “WarehouseToHome” column**

SELECT DISTINCT warehousetohome
FROM ecommercechurn

The table shows a sample of the entire result. However, notice the 126 and 127 values; they are definitely outliers and most likely wrongly entered. To fix this, we will correct the values to 26 and 27 to fall within the range of the values found in this column.

UPDATE ecommercechurn
SET warehousetohome = '27'
WHERE warehousetohome = '127'

UPDATE ecommercechurn
SET warehousetohome = '26'
WHERE warehousetohome = '126'
Our values have been replaced and are now all in the same range.

Our data has been cleaned and is now ready to be explored for insight generation.

Data Exploration
# Answering business questions.

**1. What is the overall customer churn rate?**

SELECT TotalNumberofCustomers, 
       TotalNumberofChurnedCustomers,
       CAST((TotalNumberofChurnedCustomers * 1.0 / TotalNumberofCustomers * 1.0)*100 AS DECIMAL(10,2)) AS ChurnRate
FROM
(SELECT COUNT(*) AS TotalNumberofCustomers
FROM ecommercechurn) AS Total,
(SELECT COUNT(*) AS TotalNumberofChurnedCustomers
FROM ecommercechurn
WHERE CustomerStatus = 'churned') AS Churned

The churn rate of 16.84% indicates that a significant portion of customers in the dataset have ended their association with the company.

**2. How does the churn rate vary based on the preferred login device?**

SELECT preferredlogindevice, 
        COUNT(*) AS TotalCustomers,
        SUM(churn) AS ChurnedCustomers,
        CAST(SUM (churn) * 1.0 / COUNT(*) * 100 AS DECIMAL(10,2)) AS ChurnRate
FROM ecommercechurn
GROUP BY preferredlogindevice

The preferred login device appears to have some influence on customer churn rates. Customers who prefer logging in using a computer have a slightly higher churn rate compared to customers who prefer logging in using their phones. This may indicate that customers who access the platform via a computer might have different usage patterns, preferences, or experiences that contribute to a higher likelihood of churn. Understanding these preferences can help businesses optimize their platform and user experience for different login devices, ensuring a seamless and engaging experience for customers.

**3. What is the distribution of customers across different city tiers?**

SELECT citytier, 
       COUNT(*) AS TotalCustomer, 
       SUM(Churn) AS ChurnedCustomers, 
       CAST(SUM (churn) * 1.0 / COUNT(*) * 100 AS DECIMAL(10,2)) AS ChurnRate
FROM ecommercechurn
GROUP BY citytier
ORDER BY churnrate DESC

City Tier 1 is typically a major metropolitan area with the highest level of economic development and infrastructure. These cities are usually the most populous and have significant commercial and business centers.

City Tier 2 is considered smaller or secondary urban centers compared to Tier 1 cities.

City Tier 3 is further down the hierarchy and generally refers to smaller towns or cities with a smaller population and less developed infrastructure compared to Tier 1 and 2 cities.

The result suggests that the city tier has an impact on customer churn rates. Tier 1 cities have a relatively lower churn rate compared to Tier 2 and Tier 3 cities. This could be attributed to various factors such as competition, customer preferences, or the availability of alternatives in different city tiers.

**4. Is there any correlation between the warehouse-to-home distance and customer churn?**

In order to answer this question, we will create a new column called “WarehouseToHomeRange” that groups the distance into very close, close, moderate, and far using the CASE statement.

ALTER TABLE ecommercechurn
ADD warehousetohomerange NVARCHAR(50)

UPDATE ecommercechurn
SET warehousetohomerange =
CASE 
    WHEN warehousetohome <= 10 THEN 'Very close distance'
    WHEN warehousetohome > 10 AND warehousetohome <= 20 THEN 'Close distance'
    WHEN warehousetohome > 20 AND warehousetohome <= 30 THEN 'Moderate distance'
    WHEN warehousetohome > 30 THEN 'Far distance'
END
Finding a correlation between warehouse to home and churn rate.

SELECT warehousetohomerange,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY warehousetohomerange
ORDER BY Churnrate DESC

The distance between the warehouse and the customer’s home seems to have some influence on customer churn rates. Customers residing in closer proximity to the warehouse tend to have lower churn rates, while customers living at further distances are more likely to churn. This suggests that factors such as delivery times, shipping costs, or convenience may play a role in customer retention. The company can utilize these insights to optimize its logistics and delivery strategies, ensuring better service for customers residing at further distances and implementing retention initiatives for customers in higher churn rate categories.

**5. Which is the most preferred payment mode among churned customers?**

SELECT preferredpaymentmode,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY preferredpaymentmode
ORDER BY Churnrate DESC

The most preferred payment mode among churned customers is cash on delivery. The preferred payment mode seems to have some influence on customer churn rates. Payment modes such as “Cash on Delivery” and “E-wallet” show higher churn rates, indicating that customers using these payment modes are more likely to churn. On the other hand, payment modes like “Credit Card” and “Debit Card” have relatively lower churn rates.

**6. What is the typical tenure for churned customers?**

First, we will create a new column called “TenureRange” that groups the customer tenure into 6 months, 1 year, 2 years, and more than 2 years using the CASE statement.

ALTER TABLE ecommercechurn
ADD TenureRange NVARCHAR(50)

UPDATE ecommercechurn
SET TenureRange =
CASE 
    WHEN tenure <= 6 THEN '6 Months'
    WHEN tenure > 6 AND tenure <= 12 THEN '1 Year'
    WHEN tenure > 12 AND tenure <= 24 THEN '2 Years'
    WHEN tenure > 24 THEN 'more than 2 years'
END
Finding typical tenure for churned customers

SELECT TenureRange,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY TenureRange
ORDER BY Churnrate DESC

This shows that customers who have been with the company for longer periods, specifically more than 2 years in this case, have shown a lower likelihood of churn compared to customers in shorter tenure groups.

**7. Is there any difference in churn rate between male and female customers?**

SELECT gender,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY gender
ORDER BY Churnrate DESC

Both male and female customers exhibit churn rates, with males having a slightly higher churn rate compared to females. However, the difference in churn rates between the genders is relatively small. This suggests that gender alone may not be a significant factor in predicting customer churn.

**8. How does the average time spent on the app differ for churned and non-churned customers?**
SELECT customerstatus, avg(hourspendonapp) AS AverageHourSpentonApp
FROM ecommercechurn
GROUP BY customerstatus

Both churned and staying customers have the same average hours spent on the app; this indicates that the average app usage time does not seem to be a differentiating factor between customers who churned and those who stayed.

**9. Does the number of registered devices impact the likelihood of churn?
**
SELECT NumberofDeviceRegistered,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY NumberofDeviceRegistered
ORDER BY Churnrate DESC

There seems to be a correlation between the number of devices registered by customers and the likelihood of churn. Customers with a higher number of registered devices, such as 6 or 5, exhibit higher churn rates. On the other hand, customers with fewer registered devices, such as 2 or 1, show relatively lower churn rates.

**10. Which order category is most preferred among churned customers?**

SELECT preferedordercat,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY preferedordercat
ORDER BY Churnrate DESC

The analysis suggests that different order categories have varying impacts on customer churn rates. Customers who primarily order items in the “Mobile Phone” category have the highest churn rate, indicating a potential need for targeted retention strategies for this group. On the other hand, the “Grocery” category exhibits the lowest churn rate, suggesting that customers in this category may have higher retention and loyalty.

**11. Is there any relationship between customer satisfaction scores and churn?**

SELECT satisfactionscore,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY satisfactionscore
ORDER BY Churnrate DESC

The result indicates that customers with higher satisfaction scores, particularly those who rated their satisfaction as 5, have a relatively higher churn rate compared to other satisfaction score categories. This suggests that even highly satisfied customers may still churn, highlighting the importance of proactive customer retention strategies across all satisfaction levels.

**12. Does the marital status of customers influence churn behavior?**

SELECT maritalstatus,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY maritalstatus
ORDER BY Churnrate DESC

Single customers have the highest churn rate compared to customers with other marital statuses. This indicates that single customers may be more likely to discontinue their relationship with the company. On the other hand, married customers have the lowest churn rate, followed by divorced customers.

**13. How many addresses do churned customers have on average?
**
SELECT AVG(numberofaddress) AS Averagenumofchurnedcustomeraddress
FROM ecommercechurn
WHERE customerstatus = 'stayed'

On average, customers who churned had four addresses associated with their accounts.

**14. Do customer complaints influence churned behavior?**

SELECT complainrecieved,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY complainrecieved
ORDER BY Churnrate DESC

The fact that a larger proportion of customers who stopped using the company’s services registered complaints indicates the importance of dealing with and resolving customer concerns. This is vital for decreasing the number of customers who leave and building lasting loyalty. By actively listening to customer feedback, addressing their issues, and consistently working on improving the quality of their offerings, companies can create a better overall experience for customers. This approach helps to minimize the chances of customers leaving and fosters stronger relationships with them in the long run.

**15. How does the use of coupons differ between churned and non-churned customers?
**
SELECT customerstatus, SUM(couponused) AS SumofCouponUsed
FROM ecommercechurn
GROUP BY customerstatus

The higher coupon usage among stayed customers indicates their higher level of loyalty and engagement with the company. By implementing strategies to reward loyalty, provide personalized offers, and maintain continuous engagement, the company can further leverage coupon usage as a tool to strengthen customer loyalty and increase overall customer retention.

**16. What is the average number of days since the last order for churned customers?
**
SELECT AVG(daysincelastorder) AS AverageNumofDaysSinceLastOrder
FROM ecommercechurn
WHERE customerstatus = 'churned'

The fact that churned customers have, on average, only had a short period of time since their last order indicates that they recently stopped engaging with the company. By focusing on enhancing the overall customer experience, implementing targeted retention initiatives, and maintaining continuous engagement, the company can work towards reducing churn and increasing customer loyalty.

**17. Is there any correlation between cashback amount and churn rate?**  

First, we will create a new column called “CashbackAmountRange” that groups the cashbackamount into low (less than 100), moderate (between 100 and 200), high( between 200 and 300), and very high (more than 300) using the CASE statement.

ALTER TABLE ecommercechurn
ADD cashbackamountrange NVARCHAR(50)

UPDATE ecommercechurn
SET cashbackamountrange =
CASE 
    WHEN cashbackamount <= 100 THEN 'Low Cashback Amount'
    WHEN cashbackamount > 100 AND cashbackamount <= 200 THEN 'Moderate Cashback Amount'
    WHEN cashbackamount > 200 AND cashbackamount <= 300 THEN 'High Cashback Amount'
    WHEN cashbackamount > 300 THEN 'Very High Cashback Amount'
END
Finding the correlation between cashback amount range and churned rate

SELECT cashbackamountrange,
       COUNT(*) AS TotalCustomer,
       SUM(Churn) AS CustomerChurn,
       CAST(SUM(Churn) * 1.0 /COUNT(*) * 100 AS DECIMAL(10,2)) AS Churnrate
FROM ecommercechurn
GROUP BY cashbackamountrange
ORDER BY Churnrate DESC

Customers who received moderate cashback amounts had a relatively higher churn rate, while those who received higher and very high cashback amounts exhibited lower churn rates. Customers who received lower cashback amounts also had a 100% retention rate. This suggests that offering higher cashback amounts can positively influence customer loyalty and reduce churn.

# Insight Section
- The dataset includes 5,630 customers, providing a substantial sample size for analysis.
- The overall churn rate is 16.84%, indicating significant customer attrition.
- Customers who prefer logging in with a computer have slightly higher churn rates compared to phone users, suggesting different usage patterns and preferences.
- Tier 1 cities have lower churn rates than Tier 2 and Tier 3 cities, possibly due to competition and customer preferences.
- Proximity to the warehouse affects churn rates, with closer customers showing lower churn, highlighting the importance of optimizing logistics and delivery strategies.
- “Cash on Delivery” and “E-wallet” payment modes have higher churn rates, while “Credit Card” and “Debit Card” have lower churn rates, indicating the influence of payment preferences on churn.
- Longer tenure is associated with lower churn rates, emphasizing the need for building customer loyalty early on.
- Male customers have slightly higher churn rates than female customers, although the difference is minimal.
- App usage time does not significantly differentiate between churned and non-churned customers.
- More registered devices correlate with higher churn rates, suggesting the need for consistent experiences across multiple devices.
- “Mobile Phone” order category has the highest churn rate, while “Grocery” has the lowest, indicating the importance of tailored retention strategies for specific categories.
- Highly satisfied customers (rating 5) have a relatively higher churn rate, highlighting the need for proactive retention strategies at all satisfaction levels.
- Single customers have the highest churn rate, while married customers have the lowest, indicating the influence of marital status on churn.
- Churned customers have an average of four associated addresses, suggesting higher mobility.
- Customer complaints are prevalent among churned customers, emphasizing the importance of addressing concerns to minimize churn.
- Coupon usage is higher among non-churned customers, showcasing the effectiveness of loyalty rewards and personalized offers.
- Churned customers have had a short time since their last order, indicating recent disengagement and the need for improved customer experience and retention initiatives.
- Moderate cashback amounts correspond to higher churn rates, while higher amounts lead to lower churn, suggesting the positive impact of higher cashback on loyalty.
# Recommendations to reduce customer churn rate
- Enhance the user experience for customers who prefer logging in via a computer. Conduct research to identify and address any issues they might be facing, making improvements to ensure a smoother and more enjoyable experience.
- Tailor retention strategies based on the different city tiers. Understand the preferences and needs of customers in each tier to provide targeted offerings and incentives that resonate with them.
- Optimize logistics and delivery to improve customer satisfaction. Focus on reducing delivery times, lowering shipping costs, and finding ways to make the process more convenient, especially for customers living further away.
- Simplify payment processes, particularly for options like “Cash on Delivery” and “E-wallet.” Enhance security measures and offer incentives for customers to use more reliable payment methods such as “Credit Card” and “Debit Card.”
- Improve customer support and complaint resolution. Address customer complaints promptly and effectively, providing satisfactory resolutions. Actively listen to customer feedback, make necessary improvements, and demonstrate a commitment to addressing their concerns.
- Develop targeted retention initiatives for specific order categories, such as the “Mobile Phone” category. Offer exclusive discounts, rewards, or promotions to incentivize continued engagement and loyalty.
- Ensure a consistent experience across different devices. Implement features like cross-device syncing, personalized recommendations, and easy account management to encourage usage and retention across multiple devices.
- Proactively engage and reward satisfied customers across all satisfaction levels. Regularly communicate with them through personalized messages, exclusive offers, and loyalty programs to maintain their loyalty and reduce the risk of churn.
- Consider increasing cashback incentives to retain customers, especially those who are more likely to churn. Conduct A/B testing to determine the most effective cashback levels that encourage higher customer retention rates.
- By implementing these recommendations, this company can improve customer retention, reduce churn rates, and build long-term loyalty, leading to sustainable growth and success.

