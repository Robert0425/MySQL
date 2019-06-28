員工銷售金額比(由高到低排序)
select e.employeeid,sum(od.unitprice*(1-od.discount)*od.quantity)-sum(o.freight) as total from employees e join orders o on(e.employeeid = o.employeeid) join `Order Details` od on(od.orderID = o.orderid) where year(o.orderdate)>=1997 group by e.employeeid order by total desc limit 3;

城市銷售總金額比(由高到低排序)
select c.city from customers c join orders o on(c.customerid=o.customerid) join `Order Details` od on(o.orderid=od.orderid) group by c.city order by sum(od.unitprice*(1-od.discount)*od.quantity) desc;


0. 每個辦公室的業績狀況表 
select o.officecode,sum(od.quantityOrdered*od.priceEach) as total from offices o join employees e on(o.officecode=e.officecode) join customers c on(c.salesrepemployeenumber=e.employeenumber) join orders os on(os.customernumber=c.customernumber) join orderdetails od on(od.ordernumber=os.ordernumber) group by o.officecode order by total desc;


1. 業務業績排行榜, procedure, time, orders.status = shipped
DELIMITER $$
CREATE DEFINER=`root`@`localhost` PROCEDURE `employee_grade`(IN `startdate` DATE, IN `enddate` DATE)
    NO SQL
begin
select e.employeenumber,sum(od.quantityOrdered*od.priceEach) as grade from employees e join customers c on(c.salesrepemployeenumber=e.employeenumber) join orders os on(os.customernumber=c.customernumber) join orderdetails od on(od.ordernumber=os.ordernumber) WHERE startdate<os.orderDate<enddate and os.status = 'Shipped' group by e.employeenumber order by grade desc;
end$$
DELIMITER ;


2. 熱銷商品排行榜, procedure,  time, orders.status = shipped
DELIMITER $$
CREATE DEFINER=`root`@`localhost` PROCEDURE `pd`(IN `startdate` DATE, IN `enddate` DATE)
begin
select p.productName,sum(o.quantityordered) as total from products p  JOIN orderdetails o on(p.productcode = o.productcode) join orders os on(o.orderNumber = os.orderNumber) WHERE startdate<os.orderDate<enddate and os.status = 'Shipped' group by p.productcode order by total desc; 
end$$
DELIMITER ;


3. 業務訂單取消的比例 orders.status = canceled
select count(*) into @all from orders;
select count(*) into @can from orders where status = 'Cancelled';
select concat(@can/@all*100,'%');


4.  低於商品建議售價MSRP x 95% 的訂單及其實際售價
select o.ordernumber,o.priceeach from orderdetails o where o.priceeach < 0.95*(select msrp from products where productcode = o.productcode);


5. 當訂單資料被修改及刪除的時候記錄在 log 資料表中, 時間及所有資料欄位
 create table log like orders;
ALTER TABLE `log` ADD `time` DATETIME NOT NULL AFTER `customerNumber`;
CREATE TRIGGER `update` AFTER UPDATE ON `orders`
 FOR EACH ROW BEGIN
     INSERT INTO log VALUES (NEW.ordernumber,NEW.orderdate,NEW.requireddate,NEW.shippeddate,NEW.status,NEW.comments,NEW.customerNumber,now());
END


BEGIN
     INSERT INTO log VALUES (OLD.ordernumber,OLD.orderdate,OLD.requireddate,OLD.shippeddate,OLD.status,OLD.comments,OLD.customerNumber,now());
END
