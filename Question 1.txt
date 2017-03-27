--Tom Weatherall
-- 10/3/2016
--Question #1

drop table dbo.#SubTot2008;
drop table dbo.#SubTot2007;

--temp table 2008
select sum(soh.SubTotal) as subT2008, soh.SalesPersonID, year(soh.orderdate) as FY,
case 
	when month(soh.OrderDate) >=1 and month(soh.OrderDate) <=3 and year(soh.orderdate) = '2008' then 'Q3' 
	when month(soh.OrderDate) >=4 and month(soh.OrderDate) <=6 and year(soh.orderdate) = '2008' then 'Q4' 
	when month(soh.OrderDate) >=7 and month(soh.OrderDate) <=9 and year(soh.orderdate) = '2007' then 'Q1' 
	when month(soh.OrderDate) >=10 and month(soh.OrderDate) <= 12 and year(soh.orderdate) = '2007' then 'Q2' 
	end as FQ
into #SubTot2008
from sales.SalesOrderHeader soh 
where CONVERT(varchar(10), soh.OrderDate, 20) between '2007-07-01' and '2008-06-30'
and soh.OnlineOrderFlag <> 1
group by soh.SalesPersonID, year(soh.orderdate), 
case 
	when month(soh.OrderDate) >=1 and month(soh.OrderDate) <=3 and year(soh.orderdate) = '2008' then 'Q3' 
	when month(soh.OrderDate) >=4 and month(soh.OrderDate) <=6 and year(soh.orderdate) = '2008' then 'Q4' 
	when month(soh.OrderDate) >=7 and month(soh.OrderDate) <=9 and year(soh.orderdate) = '2007' then 'Q1' 
	when month(soh.OrderDate) >=10 and month(soh.OrderDate) <= 12 and year(soh.orderdate) = '2007' then 'Q2' 
	end
order by soh.SalesPersonID, FQ


--temp table 2007
select sum(soh.SubTotal) as subT2007, soh.SalesPersonID, year(soh.orderdate) as FY, 
case 
	when month(soh.OrderDate) >=1 and month(soh.OrderDate) <=3 and year(soh.orderdate) = '2007' then 'Q3' 
	when month(soh.OrderDate) >=4 and month(soh.OrderDate)  <=6 and year(soh.orderdate) = '2007' then 'Q4' 
	when month(soh.OrderDate) >=7 and month(soh.OrderDate) <=9 and year(soh.orderdate) = '2006' then 'Q1' 
	when month(soh.OrderDate) >=10 and month(soh.OrderDate) <= 12 and year(soh.orderdate) = '2006' then 'Q2' 
	end as FQ
into #SubTot2007
from sales.SalesOrderHeader soh 
where CONVERT(varchar(10), soh.OrderDate, 20) between '2006-07-01' and '2007-06-30'
and soh.OnlineOrderFlag <> 1
group by soh.SalesPersonID, year(soh.orderdate),
case 
	when month(soh.OrderDate) >=1 and month(soh.OrderDate) <=3 and year(soh.orderdate) = '2007' then 'Q3' 
	when month(soh.OrderDate) >=4 and month(soh.OrderDate) <=6 and year(soh.orderdate) = '2007' then 'Q4' 
	when month(soh.OrderDate) >=7 and month(soh.OrderDate) <=9 and year(soh.orderdate) = '2006' then 'Q1' 
	when month(soh.OrderDate) >=10 and month(soh.OrderDate) <= 12 and year(soh.orderdate) = '2006' then 'Q2' 
	end
order by soh.SalesPersonID, FQ


--join temporary tables
select p.LastName, s8.SalesPersonID, '2008' as FY, s8.FQ as FQ, s8.subT2008 as FQSales, 
s7.subT2007 as SalesSameFQLast, (s8.subT2008 - s7.subT2007) as Change, 100*((s8.subT2008 - s7.subT2007)/s7.subT2007) as [%Change]
from Person.person p 
inner join 
#SubTot2008 s8
on s8.SalesPersonID = p.BusinessEntityID
left join #SubTot2007 s7
on s7.SalesPersonID = s8.SalesPersonID
and s7.FQ = s8.FQ
group by s8.FQ,p.LastName, s8.FY, s8.SalesPersonID, s7.subT2007, s7.FQ, s8.subT2008
order by s8.SalesPersonID
