# Microsoft SQL Data Analysis and RFM Segmentation


Conducted an in-depth data analysis project utilizing Microsoft SQL to extract valuable insights and performed Recency-Frequency-Monetary (RFM) segmentation. Leveraged Tableau for effective visualization of findings and actionable recommendations.



Below the snapshot of the Tableau Presentation:
<img width="760" alt="1" src="https://github.com/sumit-kumar-crypto/Microsoft-SQL-Data-Analysis-and-RFM-Segmentation/assets/83686292/f144bb8c-8fcb-4197-9f54-029139d535b1">
<img width="758" alt="2" src="https://github.com/sumit-kumar-crypto/Microsoft-SQL-Data-Analysis-and-RFM-Segmentation/assets/83686292/299777a6-7a2e-4282-b818-077f7e173b1b">


**Performed analysis in MS SQL is following:**


_---DATA OVERLOOK:_

select * from sales_data_sample

_---UNIQUE VALUE CHECK:_

select distinct status from sales_data_sample
select distinct year_id from sales_data_sample
select distinct COUNTRY from sales_data_sample
select distinct TERRITORY from sales_data_sample
select distinct DEALSIZE from sales_data_sample
select distinct PRODUCTLINE from sales_data_sample

_---DATA ANALYSIS_


_---Year Wise Production:_

select YEAR_ID,SUM(sales) as REVENUE
from sales_data_sample
group by YEAR_ID
ORDER BY 2 DESC

_---Grouping Sales by PRODUCTLINE:_

select PRODUCTLINE,SUM(sales) as REVENUE
from sales_data_sample
group by PRODUCTLINE
ORDER BY 2 DESC

_---Examining the number of months Sales was active in different years:_

select YEAR_ID,count(distinct(MONTH_ID)) FROM sales_data_sample
group by YEAR_ID



_---Months with highest number of REVENUE generated:_

select YEAR_ID,MONTH_ID,SUM(sales) as REVENUE,COUNT(ORDERNUMBER) AS FREQUENCY
from sales_data_sample
group by YEAR_ID,MONTH_ID
ORDER BY 3 DESC


_---Highest Revenue being generated in November and October,Let's examine the PRODUCTLINE of those months:_

select YEAR_ID,MONTH_ID,PRODUCTLINE,SUM(sales) as REVENUE,COUNT(ORDERNUMBER) AS FREQUENCY
from sales_data_sample
group by YEAR_ID,MONTH_ID,PRODUCTLINE
ORDER BY 4 DESC




_---DEALSIZE proportion according to REVENUE generated:_

select DEALSIZE,SUM(sales) as REVENUE
from sales_data_sample
group by DEALSIZE
ORDER BY 2 DESC


_---Finding out the best customer with RFM Analysis:_


DROP TABLE IF EXISTS #rfm
;with rfm as 
(
	select 
		CUSTOMERNAME, 
		sum(sales) MonetaryValue,
		avg(sales) AvgMonetaryValue,
		count(ORDERNUMBER) Frequency,
		max(ORDERDATE) last_order_date,
		(select max(ORDERDATE) from sales_data_sample) max_order_date,
		DATEDIFF(DD, max(ORDERDATE), (select max(ORDERDATE) from [dbo].[sales_data_sample])) Recency
	from sales_data_sample
	group by CUSTOMERNAME
),
rfm_calc as
(

	select r.*,
		NTILE(4) OVER (order by Recency desc) rfm_recency,
		NTILE(4) OVER (order by Frequency) rfm_frequency,
		NTILE(4) OVER (order by MonetaryValue) rfm_monetary
	from rfm r
)
select 
	c.*, rfm_recency+ rfm_frequency+ rfm_monetary as rfm_cell,
	cast(rfm_recency as varchar) + cast(rfm_frequency as varchar) + cast(rfm_monetary  as varchar)rfm_cell_string
into #rfm
from rfm_calc c

select CUSTOMERNAME , rfm_recency, rfm_frequency, rfm_monetary,
	case 
		when rfm_cell_string in (111, 112 , 121, 122, 123, 132, 211, 212, 114, 141,113,221) then 'lost_customers'  
		when rfm_cell_string in (133, 134, 143, 244, 334, 343, 344, 144) then 'slipping away, cannot lose' 
		when rfm_cell_string in (311, 411,412, 331,421) then 'new customers'
		when rfm_cell_string in (222, 223, 233, 322,234,232) then 'potential churners'
		when rfm_cell_string in (323, 333,321, 422,332, 432,423) then 'active' 
		when rfm_cell_string in (433, 434, 443, 444) then 'loyal'
	end rfm_segment

from #rfm





