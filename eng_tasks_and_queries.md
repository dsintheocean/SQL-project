**1. Count how many companies have closed.**

SELECT COUNT(id)   
FROM company   
WHERE status='closed' 

---
**2.Display the amount of funds raised for news companies in the USA. 
Use the data from the 'company' table. Sort the table in descending order of values in the 'funding_total' field.**

SELECT SUM(funding_total)  
FROM company  
WHERE category_code='news' AND country_code='USA'  
GROUP BY id  
ORDER BY SUM(funding_total) DESC  

---
**3. Find the total amount of deals for the purchase of one company by another in dollars. 
Select deals that were conducted exclusively in cash from 2011 to 2013, inclusive.**

SELECT SUM(price_amount)  
FROM acquisition  
WHERE term_code='cash'  
  AND EXTRACT(YEAR FROM acquired_at) IN (2011, 2012, 2013)  

---
**4. Display the first name, last name, and Twitter account names of people whose account names start with 'Silver'.**

SELECT first_name,  
       last_name,  
       twitter_username  
FROM PEOPLE  
WHERE twitter_username LIKE 'Silver%'  

---
**5. Display all the information about people whose Twitter account names contain the substring 'money' and whose last names start with 'K'.**

SELECT *  
FROM PEOPLE  
WHERE twitter_username LIKE '%money%'  
  AND last_name LIKE 'K%'  

---
**6. For each country, display the total amount of investments received by companies registered in that country. 
The country of registration can be determined by the country code. 
Sort the data in descending order of the total amount.**

SELECT country_code,  
       SUM(funding_total)  
FROM company  
GROUP BY country_code  
ORDER BY SUM(funding_total) DESC  

---
**7. Create a table that includes the date of the round, as well as the minimum and maximum values of the investment amount raised on that date. 
Keep only the records in the final table where the minimum investment amount is not equal to zero and is not equal to the maximum amount.**

SELECT funded_at,  
       MIN(raised_amount),  
       MAX(raised_amount)  
FROM funding_round  
GROUP BY funded_at  
HAVING MIN(raised_amount) != 0  
  AND MIN(raised_amount) != MAX(raised_amount)  

---
**8. Create a field with categories:
Assign the category 'high_activity' to funds that invest in 100 or more companies.
Assign the category 'middle_activity' to funds that invest in 20 or more companies but fewer than 100.
Assign the category 'low_activity' to funds that invest in fewer than 20 companies.
Display all fields in the 'fund' table along with the new category field.**

SELECT *,  
       CASE  
           WHEN invested_companies >= 100 THEN 'high_activity'  
           WHEN invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'  
           WHEN invested_companies < 20 THEN 'low_activity'  
       END  
FROM fund  

---
**9. For each of the categories assigned in the previous task, calculate the rounded average number of investment rounds in which the fund participated. Display the categories and the average number of investment rounds. 
Sort the table in ascending order of the average.**

SELECT ROUND(AVG(investment_rounds)),  
       CASE  
           WHEN invested_companies>=100 THEN 'high_activity'  
           WHEN invested_companies>=20 THEN 'middle_activity'  
           ELSE 'low_activity'  
       END AS activity  
FROM fund  
GROUP BY activity  
ORDER BY ROUND(AVG(investment_rounds))  

---
**10.Analyze the countries where funds that invest in startups most frequently operate. 
For each country, calculate the minimum, maximum, and average number of companies that funds from that country have invested in, founded between 2010 and 2012, inclusive. 
Exclude countries with funds that have a minimum number of invested companies equal to zero. 
Retrieve the top ten most active investing countries: sort the table by the average number of companies from highest to lowest. 
Then, add a secondary sorting by country code in lexicographic order.**

SELECT country_code,  
       MIN(invested_companies),  
       MAX(invested_companies),  
       AVG(invested_companies)  
FROM fund  
WHERE EXTRACT(YEAR FROM founded_at) BETWEEN 2010 AND 2012  
GROUP BY country_code  
HAVING MIN(invested_companies) != 0  
ORDER BY AVG(invested_companies) DESC, country_code  
LIMIT 10  

---
**11. Display the first and last names of all startup employees. 
Add a field with the name of the educational institution the employee graduated from, if this information is available.**

SELECT p.first_name,  
       p.last_name,  
       e.instituition  
FROM people AS p  
LEFT OUTER JOIN education AS e ON e.person_id=p.id  

---
**12. For each company, find the number of educational institutions that their employees have graduated from. Display the company name and the count of unique educational institution names. 
Create a top-5 list of companies based on the number of universities.**

SELECT c.name,  
       COUNT(DISTINCT e.instituition)  
FROM people AS p  
LEFT OUTER JOIN education AS e ON e.person_id=p.id  
RIGHT OUTER JOIN company AS c ON c.id=p.company_id  
GROUP BY c.id  
ORDER BY COUNT(DISTINCT e.instituition) DESC  
LIMIT 5  

---
**13. Create a list of unique names of closed companies for which the first funding round turned out to be the last.**

SELECT DISTINCT name  
FROM company  
WHERE status='closed'  
  AND funding_rounds=1  
  
---
**14. Compile a list of unique employee numbers who work for companies selected in the previous task.**

WITH  
test AS (SELECT DISTINCT cnm.name  
FROM funding_round AS fr  
FULL OUTER JOIN company AS cnm ON fr.company_id=cnm.id  
WHERE fr.is_first_round=1 AND fr.is_last_round=1 AND cnm.status='closed')  
SELECT DISTINCT p.id  
FROM test AS t  
INNER JOIN company AS c ON t.name=c.name  
INNER JOIN people AS p ON p.company_id=c.id  

---
**15. Create a table that includes unique pairs of employee numbers from the previous task and the educational institution the employee graduated from.**

WITH  
test AS (SELECT DISTINCT cnm.name  
FROM funding_round AS fr  
FULL OUTER JOIN company AS cnm ON fr.company_id=cnm.id  
WHERE fr.is_first_round=1 AND fr.is_last_round=1 AND cnm.status='closed'),  
test2 AS (SELECT DISTINCT p.id  
FROM test AS t  
INNER JOIN company AS c ON t.name=c.name  
INNER JOIN people AS p ON p.company_id=c.id)  
SELECT Distinct  
       t2.id,  
       e.instituition  
FROM test2 AS t2  
INNER JOIN education AS e ON e.person_id=t2.id  

---
**16. Count the number of educational institutions for each employee from the previous task. 
When counting, consider that some employees may have graduated from the same institution more than once.**

WITH  
test AS (SELECT DISTINCT cnm.name  
FROM funding_round AS fr  
FULL OUTER JOIN company AS cnm ON fr.company_id=cnm.id  
WHERE fr.is_first_round=1 AND fr.is_last_round=1 AND cnm.status='closed'),  
test2 AS (SELECT DISTINCT p.id  
FROM test AS t  
INNER JOIN company AS c ON t.name=c.name  
INNER JOIN people AS p ON p.company_id=c.id),  
test3 AS (SELECT  
       t2.id,  
       e.instituition  
FROM test2 AS t2  
INNER JOIN education AS e ON e.person_id=t2.id)  
SELECT id,  
       COUNT(instituition)  
FROM test3  
GROUP BY id  

---
**17. Extend the previous query and calculate the average number of educational institutions (all, not just unique ones) that employees from different companies have graduated from. 
Only one record should be output; there is no need for grouping.**

WITH  
test AS (SELECT DISTINCT cnm.name  
FROM funding_round AS fr  
FULL OUTER JOIN company AS cnm ON fr.company_id=cnm.id  
WHERE fr.is_first_round=1 AND fr.is_last_round=1 AND cnm.status='closed'),  
test2 AS (SELECT DISTINCT p.id  
FROM test AS t  
INNER JOIN company AS c ON t.name=c.name  
INNER JOIN people AS p ON p.company_id=c.id),  
test3 AS (SELECT  
       t2.id,  
       e.instituition  
FROM test2 AS t2  
INNER JOIN education AS e ON e.person_id=t2.id),  
test4 AS (SELECT  
       COUNT(instituition) AS cnt  
FROM test3  
GROUP BY id)  
SELECT AVG(cnt)  
FROM test4  

---
**18. Write a similar query: Calculate the average number of educational institutions (all, not just unique ones) that employees of Facebook have graduated from.**

WITH  
test2 AS (SELECT p.id  
FROM company AS c  
INNER JOIN people AS p ON p.company_id=c.id  
         WHERE c.name='Facebook'),  
test3 AS (SELECT  
       t2.id,  
       e.instituition  
FROM test2 AS t2  
INNER JOIN education AS e ON e.person_id=t2.id),  
test4 AS (SELECT  
       COUNT(instituition) AS cnt  
FROM test3  
GROUP BY id)  
SELECT AVG(cnt)  
FROM test4  

---
**19. Create a table with the following fields:
name_of_fund - the name of the fund.
name_of_company - the name of the company.
amount - the amount of investment raised by the company in a round.
The table will include data about companies that have had more than six significant milestones in their history, and funding rounds that took place from 2012 to 2013, inclusive.**

SELECT f.name AS name_of_fund,  
       c.name AS name_of_company,  
       fr.raised_amount AS amount  
FROM company AS c  
INNER JOIN funding_round AS fr ON fr.company_id=c.id  
LEFT OUTER JOIN investment AS i ON i.funding_round_id=fr.id  
INNER JOIN fund AS f ON f.id=i.fund_id  
WHERE c.milestones>6 AND EXTRACT(YEAR FROM fr.funded_at) IN (2012, 2013)  

---
**20. Retrieve a table with the following fields:
Purchasing company name
Deal amount
Name of the acquired company
Amount of investments made in the acquired company
A ratio that represents how many times the purchase amount exceeded the invested amount, rounded to the nearest whole number.
Exclude deals where the purchase amount is zero. If the investment amount in the acquired company is zero, exclude that company from the table.
Sort the table by deal amount from largest to smallest, and then by the name of the acquired company in lexicographical order. Limit the table to the first ten records.**

WITH  
acq AS (SELECT acquiring_company_id,  
               acquired_company_id,  
               price_amount  
        FROM acquisition  
        WHERE price_amount != 0),  
red_comp AS(SELECT c.id,  
                   c.name,  
                   c.funding_total  
           FROM company AS c  
           INNER JOIN acq AS a ON a.acquired_company_id=c.id  
           WHERE c.funding_total != 0),  
ring_comp AS(SELECT c.id,  
                    c.name  
             FROM company AS c  
             INNER JOIN acq AS a ON a.acquiring_company_id=c.id)  
SELECT DISTINCT  
       ring_comp.name AS acqring_name,  
       acq.price_amount AS deal_sum,  
       red_comp.funding_total,  
       red_comp.name AS acqred_name,  
       ROUND(acq.price_amount/red_comp.funding_total) AS diff  
FROM acq  
INNER JOIN ring_comp ON acq.acquiring_company_id=ring_comp.id  
INNER JOIN red_comp ON acq.acquired_company_id=red_comp.id  
ORDER BY deal_sum DESC, acqred_name  
LIMIT 10  

---
**21. Retrieve a table that includes the names of companies in the 'social' category that received financing from 2010 to 2013, inclusive. Ensure that the investment amount is not zero. 
Also, display the month number in which the financing round took place.**

SELECT c.name,  
       EXTRACT(MONTH FROM fr.funded_at)  
FROM company AS c  
INNER JOIN funding_round AS fr ON fr.company_id=c.id  
WHERE c.category_code='social'  
  AND EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013  
  AND fr.raised_amount != 0  
  
---
**22. Select data for the months from 2010 to 2013 when investment rounds took place. Group the data by the month number and create a table with the following fields:
Month number in which the rounds took place
The count of unique fund names from the USA that invested in that month
The count of companies acquired in that month
The total amount of deals for purchases in that month.**

WITH  
fnd AS (SELECT  
       EXTRACT(MONTH FROM fr.funded_at) AS mnth,  
       COUNT(DISTINCT(f.name)) AS cntf  
FROM company AS c  
RIGHT OUTER JOIN funding_round AS fr ON fr.company_id=c.id  
INNER JOIN investment AS i ON i.funding_round_id=fr.id  
INNER JOIN fund AS f ON f.id=i.fund_id  
WHERE f.country_code='USA'  
  AND EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013  
GROUP BY EXTRACT(MONTH FROM fr.funded_at)),  
acq AS (SELECT EXTRACT(MONTH FROM acquired_at) AS mnth,  
       COUNT(acquired_company_id) AS cnta,  
       SUM(price_amount) AS summa  
FROM acquisition  
WHERE EXTRACT(YEAR FROM acquired_at) BETWEEN 2010 AND 2013  
GROUP BY EXTRACT(MONTH FROM acquired_at))  
SELECT fnd.mnth,  
       fnd.cntf,  
       acq.cnta,  
       acq.summa  
FROM fnd  
INNER JOIN acq ON fnd.mnth=acq.mnth  

---
**23. Create a pivot table and display the average investment amount for countries where startups were registered in the years 2011, 2012, and 2013. The data for each year should be in a separate field. 
Sort the table by the average investment value for the year 2011 from highest to lowest.**

WITH  
y11 AS (SELECT country_code,  
       AVG(funding_total) AS AVG11  
FROM company  
WHERE EXTRACT(YEAR FROM founded_at)=2011  
GROUP BY country_code),  
y12 AS (SELECT country_code,  
       AVG(funding_total) AS AVG12  
FROM company  
WHERE EXTRACT(YEAR FROM founded_at)=2012  
GROUP BY country_code),  
y13 AS (SELECT country_code,  
       AVG(funding_total) AS AVG13  
FROM company  
WHERE EXTRACT(YEAR FROM founded_at)=2013  
GROUP BY country_code)  
SELECT y11.country_code,  
       y11.AVG11,  
       y12.AVG12,  
       y13.AVG13  
FROM y11  
INNER JOIN y12 ON y11.country_code=y12.country_code  
INNER JOIN y13 ON y13.country_code=y12.country_code  
ORDER BY y11.AVG11 DESC  
