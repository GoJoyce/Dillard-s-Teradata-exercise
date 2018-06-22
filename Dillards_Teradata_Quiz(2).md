
#Q2. There was one store in the database which had only 11 days in one of its months (in other words, that store/month/year combination only contained 11 days of transaction data). In what city and state was this store located?
  
	SELECT s.city, s.state, t.store, extract(YEAR FROM t.saledate) "year", 
	extract(MONTH FROM t.saledate) "month", COUNT(DISTINCT extract(DAY FROM t.saledate)) AS countday
	FROM TRNSACT t LEFT JOIN STRINFO s
	ON t.store = s.store
	GROUP BY 1, 2, 3, 4, 5
	ORDER BY countday;
	

#Q3. Which sku number had the greatest increase in total sales revenue from November to December?
  
	SELECT c.sku, SUM(case when tmonth=11 then amt Else 0 end) AS month11, SUM(case when tmonth = 12 then amt Else 0 end) AS month12
	FROM (SELECT sku, extract(MONTH FROM t.saledate) AS Tmonth, extract(DAY FROM t.saledate) AS Tday,amt 
			FROM TRNSACT t WHERE Tmonth=11 OR Tmonth=12) AS c 
	GROUP BY 1
	ORDER BY month12-month11 DESC;
	

#Q4. What vendor has the greatest number of distinct skus in the transaction table that do not exist in the skstinfo table? (Remember that vendors are listed as distinct numbers in our data set).

	SELECT b.vendor, COUNT(DISTINCT a.sku)
	FROM (SELECT DISTINCT t.sku FROM TRNSACT t LEFT JOIN SKSTINFO s ON t.sku = s.sku WHERE s.sku IS NULL) AS a,  SKUINFO AS b
	WHERE a.sku = b.sku
	GROUP BY b.vendor
	ORDER BY COUNT(DISTINCT a.sku) DESC;


#Q5. What is the brand of the sku with the greatest standard deviation in sprice? Only examine skus which have been part of over 100 transactions.

	SELECT s.brand, a.sku, a.std, a.tran
	FROM (SELECT sku,  STDDEV_POP(sprice) AS std, COUNT(sprice) AS tran
		FROM TRNSACT
		WHERE stype='P'
		GROUP BY sku 
		HAVING tran > 100) AS a JOIN SKUINFO AS s
	ON a.sku = s.sku
	ORDER BY a.std DESC
	

#Q6. What is the city and state of the store which had the greatest increase in average daily revenue (as I define it in Teradata Week 5 Exercise Guide) from November to December?

	SELECT c.store, c.city, c.state, SUM(case when tmonth=11 then amt Else 0 end)/COUNT(DISTINCT Tday) AS month11, SUM(case when 	tmonth = 12 then amt Else 0 end)/COUNT(DISTINCT Tday) AS month12
	FROM (SELECT t.store, s.city, s.state, extract(MONTH FROM t.saledate) AS Tmonth, extract(DAY FROM t.saledate) AS Tday, amt
		FROM trnsact t JOIN strinfo s 
		ON t.store=s.store
		WHERE Tmonth=11 OR Tmonth=12) AS c
	GROUP BY 1,2,3
	ORDER BY month12-month11 DESC;



#Q7. Compare the average daily revenue (as I define it in Teradata Week 5 Exercise Guide) of the store with the highest msa_income and the store with the lowest median msa_income (according to the msa_income field). In what city and state were these two stores, and which store had a higher average daily revenue?

	SSELECT state, city, store, msa_income
	FROM STORE_MSA
	WHERE store=2707 or store =3902
	
	
	
	
