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

