# Effective SQL Ch4

### BW24. CASE를 이용하여 A 제품은 구매, B 제품은 미구매 고객 찾기

```SQL
USE SalesOrdersSample;

SELECT CustomerID, CustFirstName, CustLastName
FROM Customers 
WHERE (1 = 
  (CASE WHEN CustomerID NOT IN -- 첫번째 WHEN: 구매내역 중 스케이트보드 미구매자는 0
    (SELECT Orders.CustomerID 
    FROM Orders INNER JOIN Order_Details
    ON Orders.OrderNumber = Order_Details.OrderNumber
    INNER JOIN Products
    ON Order_Details.ProductNumber = Products.ProductNumber
    WHERE Products.ProductName LIKE '%Skateboard%')
  THEN 0
  WHEN CustomerID IN -- 첫번째 WHEN: 구매내역 중 헬멧 구매자는 0
    (SELECT Orders.CustomerID 
    FROM Orders INNER JOIN Order_Details
    ON Orders.OrderNumber = Order_Details.OrderNumber
    INNER JOIN Products
    ON Order_Details.ProductNumber = Products.ProductNumber
    WHERE Products.ProductName LIKE '%Helmet%')
  THEN 0
  ELSE 1 END));
```
| CustomerID | CustFirstName | CustLastName | 
| -: | - | - | 
| 1011 | Alaina | Hallmark | 
| 1023 | Julia | Davidson | 


### BW25. 다중조건의 처리

**INNER JOIN**을 이용하여 다중조건 처리
```SQL
USE SalesOrdersSample;

SELECT C.CustomerID, C.CustFirstName, C.CustLastName
FROM Customers AS C INNER JOIN -- INNER JOIN1: 스케이트 보드 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Skateboard%') AS OSk
ON C.CustomerID = OSk.CustomerID
INNER JOIN -- INNER JOIN2: 헬멧 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Helmet%') AS OHel
ON C.CustomerID = OHel.CustomerID
INNER JOIN -- INNER JOIN3: 무릎보호대 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Knee Pads%') AS OKn
ON C.CustomerID = OKn.CustomerID
INNER JOIN -- INNER JOIN4: 글러브 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Gloves%') AS OGl
ON C.CustomerID = OGl.CustomerID;
```

| CustomerID | CustFirstName | CustLastName | 
| -: | - | - | 
| 1001 | Suzanne | Viescas | 
| 1002 | William | Thompson | 
| 1003 | Gary | Hallmark | 
| 1004 | Doug | Steele | 
| 1005 | Tom | Wickerath | 
| 1006 | John | Viescas | 
| 1007 | Ben | Clothier | 
| 1008 | Neil | Patterson | 
| 1009 | Andrew | Smith | 
| 1010 | Angel | Kennedy | 
| 1012 | Caroline | Viescas | 
| 1013 | Rachel | Patterson | 
| 1014 | Sam | Johnson | 
| 1015 | Darren | Smith | 
| 1016 | Jim | Davidson | 
| 1017 | Kathy | Johnson | 
| 1018 | David | Smith | 
| 1019 | Zach | Jameson | 
| 1020 | Joyce | Johnson | 
| 1021 | Deborah | Smith | 
| 1022 | Caleb | Viescas | 
| 1024 | Mark | Smith | 
| 1025 | Maria | Patterson | 
| 1026 | Kirk | Johnson | 
| 1027 | Luke | Patterson | 


**EXISTS**를 이용하여 다중조건 처리
```SQL
USE SalesOrdersSample;

SELECT C.CustomerID, C.CustFirstName, C.CustLastName
FROM Customers AS C 
WHERE EXISTS -- EXISTS1: 스케이트보드 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Skateboard%'
  AND Orders.CustomerID = C.CustomerID)
AND EXISTS -- EXISTS2: 헬멧 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Helmet%'
  AND Orders.CustomerID = C.CustomerID)
AND EXISTS -- EXISTS3: 무릎보호대 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Knee Pads%'
  AND Orders.CustomerID = C.CustomerID)
AND EXISTS -- EXISTS4: 글러브 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Gloves%'
  AND Orders.CustomerID = C.CustomerID);
```
| CustomerID | CustFirstName | CustLastName | 
| -: | - | - | 
| 1001 | Suzanne | Viescas | 
| 1002 | William | Thompson | 
| 1003 | Gary | Hallmark | 
| 1004 | Doug | Steele | 
| 1005 | Tom | Wickerath | 
| 1006 | John | Viescas | 
| 1007 | Ben | Clothier | 
| 1008 | Neil | Patterson | 
| 1009 | Andrew | Smith | 
| 1010 | Angel | Kennedy | 
| 1012 | Caroline | Viescas | 
| 1013 | Rachel | Patterson | 
| 1014 | Sam | Johnson | 
| 1015 | Darren | Smith | 
| 1016 | Jim | Davidson | 
| 1017 | Kathy | Johnson | 
| 1018 | David | Smith | 
| 1019 | Zach | Jameson | 
| 1020 | Joyce | Johnson | 
| 1021 | Deborah | Smith | 
| 1022 | Caleb | Viescas | 
| 1024 | Mark | Smith | 
| 1025 | Maria | Patterson | 
| 1026 | Kirk | Johnson | 
| 1027 | Luke | Patterson | 

- 미구매자 조건: NOT EXISTS

**WHERE**를 이용하여 다중조건 처리
```SQL
USE SalesOrdersSample;

SELECT C.CustomerID, C.CustFirstName, C.CustLastName
FROM Customers AS C 
WHERE C.CustomerID IN
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Skateboard%')
AND C.CustomerID IN
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Helmet%')
AND C.CustomerID IN
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Gloves%')
AND C.CustomerID IN
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Knee Pads%');
```
| CustomerID | CustFirstName | CustLastName | 
| -: | - | - | 
| 1018 | David | Smith | 
| 1002 | William | Thompson | 
| 1020 | Joyce | Johnson | 
| 1027 | Luke | Patterson | 
| 1017 | Kathy | Johnson | 
| 1004 | Doug | Steele | 
| 1008 | Neil | Patterson | 
| 1014 | Sam | Johnson | 
| 1013 | Rachel | Patterson | 
| 1026 | Kirk | Johnson | 
| 1021 | Deborah | Smith | 
| 1025 | Maria | Patterson | 
| 1005 | Tom | Wickerath | 
| 1009 | Andrew | Smith | 
| 1010 | Angel | Kennedy | 
| 1006 | John | Viescas | 
| 1024 | Mark | Smith | 
| 1012 | Caroline | Viescas | 
| 1016 | Jim | Davidson | 
| 1019 | Zach | Jameson | 
| 1007 | Ben | Clothier | 
| 1015 | Darren | Smith | 
| 1003 | Gary | Hallmark | 
| 1022 | Caleb | Viescas | 
| 1001 | Suzanne | Viescas | 

- 미구매자 조건: NOT IN



### BW26. GROUP BY, HAVING을 이용하여 관심제품 구매고객 목록 추출

```SQL
USE SalesOrdersSample;

DROP VIEW IF EXISTS CustomerProducts;
DROP VIEW IF EXISTS ProdsOfInterest;

-- CustomerProducts(고객-제품 통합 뷰)
CREATE VIEW CustomerProducts AS
SELECT DISTINCT Customers.CustomerID, Customers.CustFirstName,
  Customers.CustLastName,
       CASE WHEN Products.ProductName LIKE '%Skateboard%' THEN 'Skateboard'
              WHEN Products.ProductName LIKE '%Helmet%' THEN 'Helmet'
              WHEN Products.ProductName LIKE '%Knee Pads%' THEN 'Knee Pads'
              WHEN Products.ProductName LIKE '%Gloves%' THEN 'Gloves'
              ELSE NULL
       END AS ProductCategory
FROM Customers INNER JOIN Orders
  ON Customers.CustomerID = Orders.CustomerID
INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
INNER JOIN Products
  ON Products.ProductNumber = Order_Details.ProductNumber;

-- ProdsOfInterest(관심제품목록 뷰)
CREATE VIEW ProdsOfInterest AS
SELECT DISTINCT 
       CASE WHEN Products.ProductName LIKE '%Skateboard%' THEN 'Skateboard'
              WHEN Products.ProductName LIKE '%Helmet%' THEN 'Helmet'
              WHEN Products.ProductName LIKE '%Knee Pads%' THEN 'Knee Pads'
              WHEN Products.ProductName LIKE '%Gloves%' THEN 'Gloves'
              ELSE NULL
       END AS ProductCategory
FROM Products
WHERE ProductName LIKE '%Skateboard%'
   OR ProductName LIKE '%Helmet%'
   OR ProductName LIKE '%Knee Pads%'
   OR ProductName LIKE '%Gloves%';

-- View 확인
SELECT * FROM CustomerProducts;
SELECT * FROM ProdsOfInterest;

-- COUNT(CP.ProductCategory)를 이용하여 고객이 관심제품 중 몇개나 구매했는지 확인
SELECT CP.CustomerID, CP.CustFirstName, CP.CustLastName, COUNT(CP.ProductCategory)
FROM CustomerProducts CP
LEFT JOIN ProdsOfInterest PofI
   ON CP.ProductCategory = PofI.ProductCategory
GROUP BY CP.CustomerID, CP.CustFirstName, CP.CustLastName;

-- 고객이 관심제품을 모두 구매한 고객만 추출
SELECT CP.CustomerID, CP.CustFirstName, CP.CustLastName, COUNT(CP.ProductCategory)
FROM CustomerProducts CP
LEFT JOIN ProdsOfInterest PofI
   ON CP.ProductCategory = PofI.ProductCategory
GROUP BY CP.CustomerID, CP.CustFirstName, CP.CustLastName
HAVING COUNT(CP.ProductCategory) =
  (SELECT COUNT(ProductCategory) FROM ProdsOfInterest);
  
DROP VIEW IF EXISTS CustomerProducts;
DROP VIEW IF EXISTS ProdsOfInterest;
```
| CustomerID | CustFirstName | CustLastName | ProductCategory | 
| -: | - | - | - | 
| 1001 | Suzanne | Viescas | \N | 
| 1001 | Suzanne | Viescas | Knee Pads | 
| 1001 | Suzanne | Viescas | Helmet | 
| 1001 | Suzanne | Viescas | Gloves | 
| 1001 | Suzanne | Viescas | Skateboard | 
| 1002 | William | Thompson | \N | 
| 1002 | William | Thompson | Skateboard | 
| 1002 | William | Thompson | Knee Pads | 
| 1002 | William | Thompson | Helmet | 
| 1002 | William | Thompson | Gloves | 
| 1003 | Gary | Hallmark | Knee Pads | 
| 1003 | Gary | Hallmark | \N | 
| 1003 | Gary | Hallmark | Gloves | 
| 1003 | Gary | Hallmark | Helmet | 
| 1003 | Gary | Hallmark | Skateboard | 
- 일부만 기록함.


| ProductCategory | 
| - | 
| Helmet | 
| Skateboard | 
| Knee Pads | 
| Gloves | 


| CustomerID | CustFirstName | CustLastName | COUNT(CP.ProductCategory) | 
| -: | - | - | -: | 
| 1022 | Caleb | Viescas | 4 | 
| 1023 | Julia | Davidson | 3 | 
| 1024 | Mark | Smith | 4 | 
| 1025 | Maria | Patterson | 4 | 
| 1026 | Kirk | Johnson | 4 | 
| 1027 | Luke | Patterson | 4 | 
- 일부만 기록함.

| CustomerID | CustFirstName | CustLastName | COUNT(CP.ProductCategory) | 
| -: | - | - | -: | 
| 1001 | Suzanne | Viescas | 4 | 
| 1006 | John | Viescas | 4 | 
| 1012 | Caroline | Viescas | 4 | 
| 1022 | Caleb | Viescas | 4 | 
| 1005 | Tom | Wickerath | 4 | 
- 일부만 기록함.


### BW29. LEFT JOIN과 IS NULL을 이용한 차집합 연산
JOIN 후 특정 기간 구매자만 추출
```SQL
USE SalesOrdersSample;

SELECT Customers.CustomerID, Customers.CustFirstName, 
  Customers.CustLastName, Orders.OrderNumber, Orders.OrderDate,
  Orders.OrderTotal
FROM Customers LEFT JOIN Orders
  ON Customers.CustomerID = Orders.CustomerID
WHERE Orders.OrderDate BETWEEN CAST('2015-10-01' AS DATETIME)
  AND CAST('2015-12-31' AS DATETIME);
```
| CustomerID | CustFirstName | CustLastName | OrderNumber | OrderDate | OrderTotal | 
| -: | - | - | -: | - | -: | 
| 1027 | Luke | Patterson | 422 | 2015-11-25 | 5901.00 | 
| 1027 | Luke | Patterson | 440 | 2015-11-28 | 1695.00 | 
| 1027 | Luke | Patterson | 453 | 2015-11-30 | 1854.50 | 
| 1027 | Luke | Patterson | 512 | 2015-12-12 | 291.00 | 
| 1027 | Luke | Patterson | 520 | 2015-12-13 | 2949.88 | 
| 1027 | Luke | Patterson | 532 | 2015-12-17 | 7190.05 | 
| 1027 | Luke | Patterson | 575 | 2015-12-26 | 12368.54 | 
- 일부만 기록함.


JOIN 후 특정 기간 구매자 + 미구매자 추출
```SQL
USE SalesOrdersSample;

SELECT Customers.CustomerID, Customers.CustFirstName, 
  Customers.CustLastName, Orders.OrderNumber, Orders.OrderDate,
  Orders.OrderTotal
FROM Customers LEFT JOIN Orders
  ON Customers.CustomerID = Orders.CustomerID
WHERE (Orders.OrderDate BETWEEN CAST('2015-10-01' AS DATETIME)
    AND CAST('2015-12-31' AS DATETIME))
  OR Orders.OrderNumber IS NULL;
```
| CustomerID | CustFirstName | CustLastName | OrderNumber | OrderDate | OrderTotal | 
| -: | - | - | -: | - | -: | 
| 1027 | Luke | Patterson | 440 | 2015-11-28 | 1695.00 | 
| 1027 | Luke | Patterson | 453 | 2015-11-30 | 1854.50 | 
| 1027 | Luke | Patterson | 512 | 2015-12-12 | 291.00 | 
| 1027 | Luke | Patterson | 520 | 2015-12-13 | 2949.88 | 
| 1027 | Luke | Patterson | 532 | 2015-12-17 | 7190.05 | 
| 1027 | Luke | Patterson | 575 | 2015-12-26 | 12368.54 | 
| 1028 | Jeffrey | Tirekicker | \N | \N | \N | 
- 일부만 기록함.


특정 기간 구매기록만 JOIN함
```SQL
USE SalesOrdersSample;

SELECT Customers.CustomerID, Customers.CustFirstName, 
  Customers.CustLastName, OFiltered.OrderNumber, 
  OFiltered.OrderDate, OFiltered.OrderTotal
FROM Customers LEFT JOIN 
  (SELECT Orders.OrderNumber, Orders.CustomerID, 
    Orders.OrderDate, Orders.OrderTotal
  FROM Orders
  WHERE Orders.OrderDate BETWEEN CAST('2015-10-01' AS DATETIME)
    AND CAST('2015-12-31' AS DATETIME)) AS OFiltered
  ON Customers.CustomerID = OFiltered.CustomerID;
```
| CustomerID | CustFirstName | CustLastName | OrderNumber | OrderDate | OrderTotal | 
| -: | - | - | -: | - | -: | 
| 1027 | Luke | Patterson | 440 | 2015-11-28 | 1695.00 | 
| 1027 | Luke | Patterson | 453 | 2015-11-30 | 1854.50 | 
| 1027 | Luke | Patterson | 512 | 2015-12-12 | 291.00 | 
| 1027 | Luke | Patterson | 520 | 2015-12-13 | 2949.88 | 
| 1027 | Luke | Patterson | 532 | 2015-12-17 | 7190.05 | 
| 1027 | Luke | Patterson | 575 | 2015-12-26 | 12368.54 | 
| 1028 | Jeffrey | Tirekicker | \N | \N | \N | 
- 일부만 기록함.


