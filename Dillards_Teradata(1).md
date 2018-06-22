# Dillard-s-Teradata-exercise
#Exercise 1: Use COUNT and DISTINCT to determine how many distinct skus there are in pairs of the skuinfo, skstinfo, and trnsact tables. Which skus are common to pairs of tables, or unique to specific tables? 

	SELECT COUNT(DISTINCT sku) AS NumSku
	FROM skuinfo;


	SELECT COUNT(DISTINCT sku) AS NumSku
	FROM skstinfo;


	SELECT COUNT(DISTINCT sku) AS NumSku
	FROM trnsact;



#(b)Use COUNT to determine how many instances there are of each sku associated with each store in the skstinfo table and the trnsact table? 
  
	SELECT sku, store, COUNT(*) AS NumInstance
	FROM skstinfo
	GROUP BY sku, store
	ORDER BY NumInstance DESC;


	SELECT sku, store, COUNT(*) AS NumInstance
	FROM trnsact
	GROUP BY sku, store
	ORDER BY NumInstance DESC;


#Exercise 2: Use COUNT and DISTINCT to determine how many distinct stores there are in the strinfo, store_msa, skstinfo, and trnsact tables. 
	
	SELECT COUNT(DISTINCT store) AS NumStore
	FROM strinfo;

	SELECT COUNT(DISTINCT store) AS NumStore
	FROM store_msa;

	SELECT COUNT(DISTINCT store) AS NumStore
	FROM skstinfo;

	SELECT COUNT(DISTINCT store) AS NumStore
	FROM trnsact;
  
  
#Exercise 3: It turns out there are many skus in the trnsact table that are not in the skstinfo table. As a consequence, we will not be able to complete many desirable analyses of Dillard’s profit, as opposed to revenue, because we do not have the cost information for all the skus in the transact table (recall that profit = revenue - cost). Examine some of the rows in the trnsact table that are not in the skstinfo table; can you find any common features that could explain why the cost information is missing? 


	SELECT t.*
	FROM trnsact t LEFT JOIN skstinfo s
	ON t.sku = s.sku AND t.store = s.store
	WHERE s.sku IS NULL;



#Exercise 4: Although we can’t complete all the analyses we’d like to on Dillard’s profit, we can look at general trends. What is Dillard’s average profit per day?

	SELECT SUM(t.amt-s.cost)/COUNT(DISTINCT t.saledate) AS AvgProfit
	FROM trnsact t LEFT JOIN skstinfo s
	ON t.sku = s.sku AND t.store = s.store
	WHERE t.stype = 'P';



#Exercise 5: On what day was the total value (in $) of returned goods the greatest? On what day was the total number of individual returned items the greatest? 

	SELECT saledate, SUM(amt)
	FROM trnsact
	WHERE stype = 'R'
	GROUP BY saledate
	ORDER BY SUM(amt) DESC;


	SELECT saledate, SUM(quantity)
	FROM trnsact
	WHERE stype = 'R'
	GROUP BY saledate
	ORDER BY SUM(quantity) DESC;
  

#Exercise 6: What is the maximum price paid for an item in our database? What is the minimum price paid for an item in our database? 

	SELECT sprice
	FROM trnsact
	ORDER BY sprice DESC;


	SELECT sprice
	FROM trnsact
	ORDER BY sprice;

#Exercise 7: How many departments have more than 100 brands associated with them, and what are their descriptions? 


	SELECT s.dept AS Departments, d.deptdesc AS Description, COUNT(s.brand) AS No_of_Departments
	FROM deptinfo d RIGHT JOIN skuinfo s
	ON d.dept = s.dept
	GROUP BY s.dept, d.deptdesc
	HAVING COUNT(s.brand) > 100;


#Exercise 8: Write a query that retrieves the department descriptions of each of the skus in the skstinfotable. 

	SELECT s.sku AS skus, d.deptdesc AS No_of_Departments
	FROM skstinfo s LEFT JOIN skuinfo
	ON s.sku = skuinfo.sku
	LEFT JOIN deptinfo d
	ON skuinfo.dept = d.dept
	GROUP BY s.sku, d.deptdesc;


#Exercise 9: What department (with department description), brand, style, and color had the greatest total value of returned items? 

	SELECT d.dept, d.deptdesc, s.brand, s.style, s.color, SUM(t.amt) AS Total_Value
	FROM skuinfo s LEFT JOIN deptinfo d
	ON s.dept = d.dept
	LEFT JOIN trnsact t 
	ON s.sku = t.sku
	WHERE t.stype = 'R'
	GROUP BY d.dept, d.deptdesc, s.brand, s.style, s.color
	ORDER BY SUM(t.amt) DESC;


#Exercise 10: In what state and zip code is the store that had the greatest total revenue during the time period monitored in our dataset? 

	SELECT s.state, s.zip, SUM(t.amt) AS Total_Revenue
	FROM store_msa s LEFT JOIN trnsact t
	ON s.store = t.store
	WHERE t.stype = 'R'
	GROUP BY s.state, s.zip
	ORDER BY SUM(t.amt) DESC;
  
  
#Exercise 11: What is the deptdesc of the departments that have the top 3 greatest numbers of skus from the skuinfo table associated with them?

	SELECT s.sku, d.deptdesc, COUNT(skuinfo.sku)
	FROM skstinfo s RIGHT JOIN skuinfo
	ON s.sku = skuinfo.sku
	LEFT JOIN deptinfo d
	ON skuinfo.dept = d.dept
	GROUP BY s.sku, d.deptdesc
	ORDER BY COUNT(skuinfo.sku) DESC;


#Exercise 12: The store_msa table provides population statistics about the geographic location around a store. Using one query to retrieve your answer, how many MSAs are there within the state of North Carolina (abbreviated “NC”), and within these MSAs, what is the lowest population level (msa_pop) and highest income level (msa_income)?

	SELECT COUNT(msa), MIN(msa_pop), MAX(msa_income)
	FROM store_msa
	WHERE state = 'NC';


#Exercise 13: What is the suggested retail price of all the skus in the “reebok” department with the “skechers” brand and a “wht/saphire” color?

	SELECT DISTINCT sk.retail
	FROM skstinfo sk JOIN skuinfo s
	ON sk.sku = s.sku
	JOIN deptinfo d
	ON s.dept = d.dept
	WHERE s.deptdesc = 'REEBOK' AND s.brand = 'skechers' AND s.color = 'wht/saphire';
