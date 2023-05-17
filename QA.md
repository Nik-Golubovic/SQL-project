What are your risk areas? Identify and describe them.
-The largest risk areas are the duplicate fullvisitorid values in the all_sessions table and as well as the duplicates in the analytics table. The analytics table has a vast amount of data in it and to clean it all to get a primary key would take up most of the allotted time for this project so whenever we are dealing with data from analytics we will need to be mindful that there are still duplicates and we should take care when selecting data from there. The fullvisitorid column in all_sessions will need to be cleaned, toherwise we will not be able to retrive data from there properly. This is a massive risk as we will almost certainly need to delete some rows that contain relevant data, but without a resource to gain more information on how this data was retrieved we will need to make assumptions on what is considered accurate and what isn't. 


QA Process:
Below we will go over each of the 5 tables and describe what we did to ensure that our cleaning process was effective.

Sales Report Table
	-select distinct(productssku) as sku from sales_report --This checks to ensure that all of the products skus are unique in the table as this is what we will use for our primary key.

-- next we will check to see if any product names are null. SKU code is primary key so cannot be null by default.
	- select name from sales_report
		where name = 'null'

Sales By SKU Table
  --We check to see if productsku is unique.
	- select distinct(productsku) as sku from sales_by_sku

  --Ensure that the productsku and total_ordered columns match with the same columns in the sales_report table
	-select * from sales_report s
		join sales_by_sku b
		on s.productssku = b.productsku
		where s.total_ordered = b.total_ordered
	-this showed that there are an extra 9 skus in the 	sales_by_sku table that dont exist in the sales_report table
  --These are the 9 SKU's that are not in the sales_report table. We just need to be cognizant of using these SKUs for any analysis going forward. The risk for that is low.
  SELECT sbs.*
    FROM sales_by_sku AS sbs
    LEFT JOIN sales_report AS sr ON sbs.productsku = sr.productsku
    WHERE sr.productsku IS NULL;

Products Table
    --The primary key for this table is productsku, so we know that there are no duplicates in that. 
    --This check that the orderedquantity is greater than the stocklevels. There are 15 SKU's where that is the case, but since the products table seems like it keeps a running tally the orderedquantity for all time we can say that the risk for this is low.
    SELECT *
      FROM products
      WHERE orderedquantity > stocklevels;
    --We check the max restockingleadtime as below. The result is 42 days which seems reasonable.
    SELECT MAX(restockingleadtime) AS max_restockingtime
      FROM products;
     
All Sessions Table
--First we will drop all columns that seem to have only null values or that are duplicates of other columns. These columns won't be used for any calcualtions so the risk here is low.
  SELECT *
    FROM all_sessions
    WHERE columnID IS NOT NULL;
 --Next we will check for duplicates of fullvisitorid as we will use this as our primary key
 -SELECT
    fullvisitorid,
    SUM(1) AS count
    FROM all_sessions
    GROUP BY 1
    HAVING count > 1
  --We will need to remove the duplicates of fullvisitorid. If all of the columns are identical we can safely remove the duplicates, but some of the columns are not identical and we will need to go on a case by case basis to determine which row is most appropriate for deletion. This is a high risk process as     we are only capable of estimating which rows are valid inputs and which are duplicates at this point. 
  --We can see that all of the columns that have prices or transaction will need to be divided by 1000000 to get a correct dollar format. 
    -UPDATE all_sessions SET productprice = ROUND(productprice / 1000000.0, 			2)::numeric(10,2);
  --We can check that the maximum product price is a realistic value using the below. This returns 298 USD which is an acceptable price for a "Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel"
  -SELECT v2productname, productprice
    FROM all_sessions
    WHERE productprice = (SELECT MAX(productprice) FROM all_sessions);
   
Analytics
--The revenue column is not usable in its current form. It is not a function of unitprice * product quantity as it seems to have the tax included from the sale so we are going to drop it. 
	-SELECT *
    FROM analytics
    WHERE productrevenue = productprice * productquantity;
 





































