# Effective SQL Ch4

### BW24. CASE를 이용하여 A 제품은 구매, B 제품은 미구매 고객 찾기

```SQL
USE SalesOrdersSample;

SELECT CustomerID, CustFirstName, CustLastName
FROM Customers 
WHERE (1 = 
  (CASE WHEN CustomerID NOT IN --첫번째 WHEN: 구매내역 중 스케이트보드 미구매자는 0
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

### BW25. 다중조건의 처리

**INNER JOIN**을 이용하여 다중조건 처리
```SQL
USE SalesOrdersSample;

SELECT C.CustomerID, C.CustFirstName, C.CustLastName
FROM Customers AS C INNER JOIN --INNER JOIN1: 스케이트 보드 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Skateboard%') AS OSk
ON C.CustomerID = OSk.CustomerID
INNER JOIN --INNER JOIN2: 헬멧 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Helmet%') AS OHel
ON C.CustomerID = OHel.CustomerID
INNER JOIN --INNER JOIN3: 무릎보호대 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Knee Pads%') AS OKn
ON C.CustomerID = OKn.CustomerID
INNER JOIN --INNER JOIN4: 글러브 구매고객 JOIN
  (SELECT DISTINCT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Gloves%') AS OGl
ON C.CustomerID = OGl.CustomerID;
```
**EXISTS**를 이용하여 다중조건 처리
```SQL
USE SalesOrdersSample;

SELECT C.CustomerID, C.CustFirstName, C.CustLastName
FROM Customers AS C 
WHERE EXISTS --EXISTS1: 스케이트보드 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Skateboard%'
  AND Orders.CustomerID = C.CustomerID)
AND EXISTS --EXISTS2: 헬멧 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Helmet%'
  AND Orders.CustomerID = C.CustomerID)
AND EXISTS --EXISTS3: 무릎보호대 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Knee Pads%'
  AND Orders.CustomerID = C.CustomerID)
AND EXISTS --EXISTS4: 글러브 구매자
  (SELECT Orders.CustomerID
  FROM Orders INNER JOIN Order_Details
  ON Orders.OrderNumber = Order_Details.OrderNumber
  INNER JOIN Products 
  ON Products.ProductNumber = Order_Details.ProductNumber
  WHERE Products.ProductName LIKE '%Gloves%'
  AND Orders.CustomerID = C.CustomerID);
```
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
- 미구매자 조건: NOT IN



### BW26. GROUP BY, HAVING을 이용하여 관심제품 구매고객 목록 추출

```SQL
USE SalesOrdersSample;

-- View 삭제
/*    
DROP VIEW IF EXISTS CustomerProducts;
DROP VIEW IF EXISTS ProdsOfInterest;
*/

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
```

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




