Cleaning files to do list

**sales_report**

⦁	check to see if all skus are unique	

- select * from sales_report
	-select distinct(productssku) as sku from sales_report
 
 ⦁	check to see if any name field is null. SKU code is primary key so cannot be null by default.
	- select name from sales_report
		where name = 'null'

**sales_by_sku**

⦁	ensure all rows in productsku are unique
	- select * from sales_by_sku
	- select distinct(productsku) as sku from sales_by_sku

⦁	ensure that the productsku and total_ordered columns match with the same columns in the sales_report table
	-select * from sales_report s
		join sales_by_sku b
		on s.productssku = b.productsku
		where s.total_ordered = b.total_ordered
	
	-this showed that there are an extra 9 skus in the 	sales_by_sku table that dont exist in the sales_report 	table

**products**

⦁	set null values to in sentimentscore and sentimentmagnitude to zero if null
	-UPDATE products
		SET sentimentmagnitude = COALESCE(sentimentmagnitude, 0)
		WHERE sentimentmagnitude IS NULL

⦁	set all sentimentscore values to positive
	- UPDATE products 
		SET sentimentscore = ABS(sentimentscore) 
		WHERE sentimentscore < 0;

**analytics**

⦁		convert column visitnumber into integer
	-ALTER TABLE analytics 
        ALTER COLUMN visitnumber TYPE INT USING 			     		visitnumber::integer;

⦁	 convert the unit price into real price with two decimals
	- UPDATE analytics
		SET unit_price = ROUND(unit_price / 1000000.0, 				2)::numeric(10,2);

⦁	convert the varchar string in visitstarttime into a usable time format
	- UPDATE analytics
			SET visitstarttime = 									TO_CHAR(TO_TIMESTAMP(visitstarttime::bigint),     			'YYYY-MM-DD HH24:MI:SS');

⦁	delete userid column as it does not link to anything and has no values as well as socialengagementtype
	- ALTER TABLE analytics DROP COLUMN userid;
	- ALTER TABLE analytics DROP COLUMN socialengagementtype;

⦁	convert bounces and units_sold column 'null's to have a zero value instead 
	- UPDATE analytics
		SET bounces = COALESCE(bounces, 0)
		WHERE bounces IS NULL;
	-UPDATE analytics
		SET units_sold = COALESCE(units_sold, 0)
		WHERE bounces IS NULL;

⦁	the revenue column is not usable in its current form. The revenue has the tax included from the sale so it is not able to be calculated by just the units_sold * unit_price columns so we are going to drop it. 
	-ALTER TABLE analytics DROP COLUMN revenue;

⦁	change visitid column from varchar to int
	-UPDATE analytics SET visitid = CAST(visitid AS INT);

⦁	delete all the rows that have no units_sold as they are unnecassary for our analysis.

⦁	replace null values with zero in timeonsite
	-UPDATE analytics
		SET timeonsite = COALESCE(timeonsite, 0)
		WHERE timeonsite IS NULL;

**all_sessions**

⦁	figure out which columns are redundant and delete
	
	-ALTER TABLE all_sessions DROP COLUMN searchkeyword;
	-ALTER TABLE all_sessions DROP COLUMN productrefundamount;
	-ALTER TABLE all_sessions DROP COLUMN itemrevenue
	-ALTER TABLE all_sessions DROP COLUMN itemquantity
	-transactionrevenue is a duplicate of 		totaltransactionrevenueadmin
	-remove transactions and productquantity
		-ALTER TABLE your_table
			DROP COLUMN transactions,
			DROP COLUMN productquantity,
	- drop productrevenue
	-alter table all_sessions drop column productrevenue

⦁	delete all duplicates of fullvisitorid where the total transactionrevenue is null

⦁	Remove null values from transactionid
	
	-UPDATE all_sessions
		SET transactionid = COALESCE(transactionid, 'NA')
		WHERE transactionid IS NULL;

⦁	update totaltransactionrevenue where = null
	
	-UPDATE all_sessions
		SET totaltransactionrevenue = 						     COALESCE(totaltransactionrevenue, 0)
		WHERE totaltransactionrevenue IS NULL;

⦁	update timeonsite to remove nulls
	
	-UPDATE all_sessions
		SET timeonsite = COALESCE(timeonsite, 0)
		WHERE timeonsite IS NULL

⦁	fix productprice
	
	-UPDATE all_sessions
		SET productprice = ROUND(productprice / 1000000.0, 			2)::numeric(10,2);

