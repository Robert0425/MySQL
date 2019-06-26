員工銷售金額比(由高到低排序)
select e.employeeid,sum(od.unitprice*(1-od.discount)*od.quantity)-count(o.orderid)*o.freight as total from employees e join orders o on(e.employeeid = o.employeeid) join `Order Details` od on(od.orderID = o.orderid) where year(o.orderdate)>=1997 group by e.employeeid order by total desc limit 3;

城市銷售總金額比(由高到低排序)
select c.city from customers c join orders o on(c.customerid=o.customerid) join `Order Details` od on(o.orderid=od.orderid) group by c.city order by sum(od.unitprice*(1-od.discount)*od.quantity) desc;
