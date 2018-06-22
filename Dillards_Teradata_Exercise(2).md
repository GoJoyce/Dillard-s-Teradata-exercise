#Database ua_dillards
	
#Exercise 1. How many distinct dates are there in the saledate column of the transaction table for each month/year combination in the database? 

	SELECT DISTINCT (extract(year from saledate) || extract(month from saledate)) as ndate
	FROM trnsact
	ORDER by ndate; 
	
	
#Exercise 2. Use a CASE statement within an aggregate function to determine which sku had the greatest total sales during the combined summer months of June, July, and August. 

	SELECT DISTINCT sku
		,sum(case when extract(month from saledate) = 6 then amt end) as jun
		,sum(case when extract(month from saledate) = 7 then amt end) as july
		,sum(case when extract(month from saledate) = 8 then amt end) as aug
		,(jun + july + aug) as total
	FROM trnsact  
	GROUP by sku
	ORDER by total desc;
	
	
	
#Exercise 3. How many distinct dates are there in the saledate column of the transaction table for each month/year/store combination in the database? Sort your results by the number of days per combination in ascending order. 

	SELECT DISTINCT (extract(month from saledate) || extract(year from saledate) || store) as ndate ,count(dates) as num
	FROM trnsact 
	GROUP by ndate
	ORDER by num;
	
	
#Exercise 4. What is the average daily revenue for each store/month/year combination in
the database? Calculate this by dividing the total revenue for a group by the number of
sales days available in the transaction table for that group. 

	SELECT DISTINCT (extract(MONTH from saledate) || extract(YEAR from saledate) || store) as ndate
				 ,(sum(amt)/count(DISTINCT saledate)) as dailyrev
	FROM trnsact 
	GROUP by ndate
	WHERE stype = 'p' 
		and oreplace(dates, ' ', '') not like '%82005%'
	HAVING COUNT(DISTINCT saledate) > 20 			 
	ORDER by dailyrev;
	
	
#Exercise 5. What is the average daily revenue brought in by Dillard’s stores in areas of high, medium, or low levels of high school education? 

	SELECT CASE 
			 when msa_high >= 50 and msa_high <= 60 then 'low' 
			 when msa_high >  60 and msa_high <= 70 then 'medium'
			 when msa_high >  70 then 'high'
			 END as edu_lvl,(sum(rev)/sum(nday)) as avgrev     
	FROM store_msa 
	LEFT JOIN (SELECT DISTINCT (extract(MONTH from saledate) || extract(YEAR from saledate)) as ndate ,SUM(amt)as rev, store, COUNT(DISTINCT saledate) as nday
		FROM trnsact 
		GROUP by ndate, store
		WHERE stype = 'p' and oreplace(ndate, ' ', '') not like '%82005%' 
		HAVING nday > 20) as rev 
	ON store_msa.store = rev.store
	GROUP by edu_lvl;
	
	
#Exercise 6. Compare the average daily revenues of the stores with the highest median msa_income and the lowest median msa_income. In what city and state were these stores, and which store had a higher average daily revenue?

#high level

	SELECT (sum(rev)/sum(nday)) as avgrev, city, state	
	FROM(select top 1 city, state, msa_income, store
		from store_msa
		order by msa_income desc)as high_lvl 
	LEFT JOIN (SELECT DISTINCT (extract(MONTH from saledate) || extract(YEAR from saledate)) as ndate,
										 SUM(amt)as rev, store, COUNT(DISTINCT saledate) as nday
			FROM trnsact 
			GROUP by ndate, store
			WHERE stype = 'p' and oreplace(ndate, ' ', '') not like '%82005%' 
			HAVING nday > 20) as rev 
	ON high_lvl.store = rev.store
	GROUP by state, city;
	
	
	
#low level

	SELECT (sum(rev)/sum(nday)) as avgrev, city, state	
	FROM(SELECT top 1 city, state, msa_income, store
		from store_msa
		order by msa_income asc)as low_lvl 
	LEFT JOIN (SELECT DISTINCT (extract(MONTH from saledate) || extract(YEAR from saledate)) as ndate,
										 SUM(amt)as rev, store, COUNT(DISTINCT saledate) as nday
			FROM trnsact 
			GROUP by ndate, store
			WHERE stype = 'p' and oreplace(ndate, ' ', '') not like '%82005%' 
			HAVING nday > 20) as rev 
	ON low_lvl.store = rev.store
	GROUP by state, city;



#Exercise 7: What is the brand of the sku with the greatest standard deviation in sprice? Only examine skus that have been part of over 100 transactions. 


	SELECT s.brand, r.std
	FROM(SELECT stddev_samp(sprice) as std, sku
		FROM trnsact
		GROUP by sku
		HAVING sum(quantity) > 100
		WHERE stype = 'p' and oreplace((extract(MONTH from saledate) || extract(YEAR from saledate)), ' ', '') not like '%82005%' ) as r
	LEFT JOIN skuinfo s
	ON s.sku = r.sku
	ORDER by r.std desc;



#Exercise 8: Examine all the transactions for the sku with the greatest standard deviation in sprice, but only consider skus that are part of more than 100 transactions. 



#Exercise 9: What was the average daily revenue Dillard’s brought in during each month of the year? 


	SELECT DISTINCT (extract(MONTH from saledate) || ' /' || extract(YEAR from saledate)) as ndate
			,(SUM(amt)/COUNT(distinct saledate)) as dailyrev
	FROM trnsact 
	GROUP by ndate
	WHERE stype = 'p'
	HAVING COUNT(DISTINCT saledate) > 20 
	ORDER by dailyrev desc;
	
#Exercise 10: Which department, in which city and state of what store, had the greatest % increase in average daily sales revenue from November to December? 


	SELECT s.store, s.city, s.state, d.deptdesc, 
	       SUM(CASE when extract(month from saledate)=11 then amt end) as November,
	       COUNT(DISTINCT (CASE WHEN EXTRACT(MONTH from saledate) ='11' then saledate END)) as Nov_numdays, 
	       SUM(CASE when extract(month from saledate)=12 then amt end) as December,
	       COUNT(DISTINCT (CASE WHEN EXTRACT(MONTH from saledate) ='12' then saledate END)) as Dec_numdays, 
	       ((December/Dec_numdays)-(November/Nov_numdays))/(November/Nov_numdays)*100 AS bump
	FROM trnsact t 
	JOIN strinfo s ON t.store=s.store 
	JOIN skuinfo si ON t.sku=si.sku 
	JOIN deptinfo d ON si.dept=d.dept
	WHERE t.stype='P' and t.store||EXTRACT(YEAR from t.saledate)||EXTRACT(MONTH from t.saledate) IN (SELECT store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate)
		FROM trnsact
		GROUP BY store, EXTRACT(YEAR from saledate), EXTRACT(MONTH from saledate)
		HAVING COUNT(DISTINCT saledate)>= 20)
	GROUP BY s.store, s.city, s.state, d.deptdesc 
	ORDER BY bump DESC;
	
	
#Exercise 11: What is the city and state of the store that had the greatest decrease in average daily revenue from August to September? 



	SELECT s.store, s.city, s.state, d.deptdesc, 
	    SUM(CASE when extract(month from saledate)=8 then amt end) as August,
	    COUNT(DISTINCT (CASE WHEN EXTRACT(MONTH from saledate) ='8' then saledate END)) as Aug_numdays, 
	    SUM(CASE when extract(month from saledate)=9 then amt end) as September,
	    COUNT(DISTINCT (CASE WHEN EXTRACT(MONTH from saledate) ='9' then saledate END)) as Sep_numdays, 
	    ((September/Sep_numdays)-(August/Aug_numdays))/(August/Aug_numdays)*100 AS bump
	FROM trnsact t JOIN strinfo s ON t.store=s.store 
	JOIN skuinfo sk ON t.sku=sk.sku 
	JOIN deptinfo d ON sk.dept=d.dept
	WHERE t.stype='P' and t.store||EXTRACT(YEAR from t.saledate)||EXTRACT(MONTH from t.saledate) IN (SELECT store||EXTRACT(YEAR from saledate)||EXTRACT(MONTH from saledate)
		FROM trnsact
		GROUP BY store, EXTRACT(YEAR from saledate), EXTRACT(MONTH from saledate)
		HAVING COUNT(DISTINCT saledate)>= 20)
	GROUP BY s.store, s.city, s.state, d.deptdesc
	ORDER BY bump ASC;

