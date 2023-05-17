**Question 1: What ProductSKUs are currently unused?**

**SQL Queries: **

select * from products
where orderedquantity = 0 and stocklevels = 0

select avg(restockingleadtime) from products
where orderedquantity = 0 and stocklevels = 0

Answer: There is a list of 190 unused SKU's in the system. These repersent items that are niether in stock nor are currently on order. With an average restocking lead time of 12 days, it is safe to say that these items can be removed from the system as there is no market for them, and if there is an order in the future we can have stock built up within 2 weeks.  



**Question 2: What products are currently stocked that are not being ordered?**

**SQL Queries:**

select s.*, p.* from sales_by_sku s
join products p
on p.productsku = s.productsku
where s.total_ordered = 0

Answer: This is a list of 150 productSKUs that have not been ordered in the time that the sales report was generated. These are items that should be taken a closer look at as to why they haven't been ordered. If they are slow moving products, we should look to discountinue thier sale to free up more space or look to have the sales teams push these particular products. 



**Question 3: Where is most of the traffic on the website coming from?**

**SQL Queries:**

SELECT channelgrouping, COUNT(DISTINCT visitid) AS count
FROM analytics
GROUP BY channelgrouping;


Answer: Most of the traffic on the website is coming from refferals, with a smaller cohort coming from organic searches. This information can be used to better focus our marketing capital to either increase our efforts in social media or other forms of passive advertisement, or to double down and try to increase traffic through further referrals, as that is the bulk of our traffic volume.



**Question 4: In preperation for the holiday season, what products are typically sold in november or december?**

**SQL Queries:**

SELECT p.name, a.productsku, SUM(an.units_sold) AS units_sold
FROM all_sessions AS a
JOIN analytics AS an ON a.fullvisitorid = an.fullvisitorid
JOIN products AS p ON a.productsku = p.productsku
WHERE EXTRACT(MONTH FROM a.date) IN (11, 12)
GROUP BY p.name, a.productsku
ORDER BY units_sold DESC;


Answer: This query returns a list of 10 products that people purchase in november and december in what quantities so that we can be prepared and have stock readily available.
