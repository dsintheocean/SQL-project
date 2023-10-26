**1. Посчитайте, сколько компаний закрылось.**

SELECT COUNT(id)   
FROM company   
WHERE status='closed' 

---
**2.Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total.**

SELECT SUM(funding_total)  
FROM company  
WHERE category_code='news' AND country_code='USA'  
GROUP BY id  
ORDER BY SUM(funding_total) DESC  

---
**3. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.**

SELECT SUM(price_amount)  
FROM acquisition  
WHERE term_code='cash'  
  AND EXTRACT(YEAR FROM acquired_at) IN (2011, 2012, 2013)  

---
**4. Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.**

SELECT first_name,  
       last_name,  
       twitter_username  
FROM PEOPLE  
WHERE twitter_username LIKE 'Silver%'  

---
**5. Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.**

SELECT *  
FROM PEOPLE  
WHERE twitter_username LIKE '%money%'  
  AND last_name LIKE 'K%'  

---
**6. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.**

SELECT country_code,  
       SUM(funding_total)  
FROM company  
GROUP BY country_code  
ORDER BY SUM(funding_total) DESC  

---
**7. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.**

SELECT funded_at,  
       MIN(raised_amount),  
       MAX(raised_amount)  
FROM funding_round  
GROUP BY funded_at  
HAVING MIN(raised_amount) != 0  
  AND MIN(raised_amount) != MAX(raised_amount)  

---
**8. Создайте поле с категориями:
Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
Отобразите все поля таблицы fund и новое поле с категориями.**

SELECT *,  
       CASE  
           WHEN invested_companies >= 100 THEN 'high_activity'  
           WHEN invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'  
           WHEN invested_companies < 20 THEN 'low_activity'  
       END  
FROM fund  

---
**9. Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.**

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
**10. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. 
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. 
Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.**

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
**11. Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.**

SELECT p.first_name,  
       p.last_name,  
       e.instituition  
FROM people AS p  
LEFT OUTER JOIN education AS e ON e.person_id=p.id  

---
**12. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.**

SELECT c.name,  
       COUNT(DISTINCT e.instituition)  
FROM people AS p  
LEFT OUTER JOIN education AS e ON e.person_id=p.id  
RIGHT OUTER JOIN company AS c ON c.id=p.company_id  
GROUP BY c.id  
ORDER BY COUNT(DISTINCT e.instituition) DESC  
LIMIT 5  

---
**13. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.**

SELECT DISTINCT name  
FROM company  
WHERE status='closed'  
  AND funding_rounds=1  
  
---
**14. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.**

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
**15. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.**

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
**16. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.**

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
**17. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.**

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
**18. Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Facebook.**

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
**19. Составьте таблицу из полей:
name_of_fund — название фонда;
name_of_company — название компании;
amount — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.**

SELECT f.name AS name_of_fund,  
       c.name AS name_of_company,  
       fr.raised_amount AS amount  
FROM company AS c  
INNER JOIN funding_round AS fr ON fr.company_id=c.id  
LEFT OUTER JOIN investment AS i ON i.funding_round_id=fr.id  
INNER JOIN fund AS f ON f.id=i.fund_id  
WHERE c.milestones>6 AND EXTRACT(YEAR FROM fr.funded_at) IN (2012, 2013)  

---
**20. Выгрузите таблицу, в которой будут такие поля:
название компании-покупателя;
сумма сделки;
название компании, которую купили;
сумма инвестиций, вложенных в купленную компанию;
доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы. 
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.**

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
**21. Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.**

SELECT c.name,  
       EXTRACT(MONTH FROM fr.funded_at)  
FROM company AS c  
INNER JOIN funding_round AS fr ON fr.company_id=c.id  
WHERE c.category_code='social'  
  AND EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013  
  AND fr.raised_amount != 0  
  
---
**22. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
номер месяца, в котором проходили раунды;
количество уникальных названий фондов из США, которые инвестировали в этом месяце;
количество компаний, купленных за этот месяц;
общая сумма сделок по покупкам в этом месяце.**

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
**23. Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.**

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
