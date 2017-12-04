# Effective SQL Ch5

### BW30. UNION을 이용한 GROUPING
```SQL
USE item30example;

SELECT Color, Dimension, SUM(Quantity) AS TotalQuantity
FROM Inventory
GROUP BY Color, Dimension
UNION
SELECT Color, NULL AS Dimension, SUM(Quantity) AS TotalQuantity
FROM Inventory 
GROUP BY Color
UNION
SELECT NULL, Dimension, SUM(Quantity)
FROM Inventory 
GROUP BY Dimension
UNION
SELECT NULL, NULL, SUM(Quantity)
FROM Inventory;
```

| Color | Dimension | TotalQuantity | 
| - | - | -: | 
| Blue | L | 5 | 
| Blue | M | 20 | 
| Red | L | 10 | 
| Red | M | 15 | 
| Blue | \n | 25 | 
| Red | \n | 25 | 
| \n | L | 15 | 
| \n | M | 35 | 
| \n | \n | 50 | 



### BW31. GROUP BY의 키를 최소화하기
```SQL
USE SalesOrdersSample;

SELECT c.CustomerID, c.CustFirstName, c.CustLastName, c.CustState,
  MAX(o.OrderDate) AS LastOrderDate, COUNT(o.OrderNumber) AS OrderCount,
  SUM(o.OrderTotal) AS TotalAmount
FROM Customers AS c
LEFT JOIN Orders AS o
  ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID;
```
| CustomerID | CustFirstName | CustLastName | CustState | LastOrderDate | OrderCount | TotalAmount | 
| -: | - | - | - | - | -: | -: | 
| 1020 | Joyce | Johnson | WA | 2016-02-28 | 37 | 211354.30 | 
| 1021 | Deborah | Smith | TX | 2016-02-28 | 42 | 178263.45 | 
| 1022 | Caleb | Viescas | CA | 2016-02-24 | 20 | 15581.77 | 
| 1023 | Julia | Davidson | WA | 2016-02-27 | 25 | 121937.66 | 
| 1024 | Mark | Smith | TX | 2016-02-24 | 41 | 201535.69 | 
| 1025 | Maria | Patterson | TX | 2016-02-28 | 41 | 229550.43 | 
| 1026 | Kirk | Johnson | TX | 2016-02-28 | 35 | 200233.15 | 
| 1027 | Luke | Patterson | OR | 2016-02-21 | 33 | 178192.81 | 
| 1028 | Jeffrey | Tirekicker | WA | \n | 0 | \n | 
- 일부만 기록
- 이 쿼리는 MySQL, PostgreSQL에서만 작동

---
```SQL
USE SalesOrdersSample;

SELECT c.CustomerID, c.CustFirstName, c.CustLastName, c.CustState,
  o.LastOrderDate, o.OrderCount, o.TotalAmount
FROM Customers AS c
LEFT JOIN 
   (SELECT t.CustomerID, MAX(t.OrderDate) AS LastOrderDate,
    COUNT(t.OrderNumber) AS OrderCount, SUM(t.OrderTotal) AS TotalAmount
    FROM Orders AS t
    GROUP BY t.CustomerID) AS o
  ON c.CustomerID = o.CustomerID
ORDER BY c.CustomerID;
```
| CustomerID | CustFirstName | CustLastName | CustState | LastOrderDate | OrderCount | TotalAmount | 
| -: | - | - | - | - | -: | -: | 
| 1020 | Joyce | Johnson | WA | 2016-02-28 | 37 | 211354.30 | 
| 1021 | Deborah | Smith | TX | 2016-02-28 | 42 | 178263.45 | 
| 1022 | Caleb | Viescas | CA | 2016-02-24 | 20 | 15581.77 | 
| 1023 | Julia | Davidson | WA | 2016-02-27 | 25 | 121937.66 | 
| 1024 | Mark | Smith | TX | 2016-02-24 | 41 | 201535.69 | 
| 1025 | Maria | Patterson | TX | 2016-02-28 | 41 | 229550.43 | 
| 1026 | Kirk | Johnson | TX | 2016-02-28 | 35 | 200233.15 | 
| 1027 | Luke | Patterson | OR | 2016-02-21 | 33 | 178192.81 | 
| 1028 | Jeffrey | Tirekicker | WA | \n | 0 | \n | 
- 일부만 기록
- 모든 DBMS에 적용 가능한 쿼리

---
### BW32. GROUP BY, HAVING을 이용하여 복잡한 문제를 해결하자

15년도 카테고리 별 평균 판매액보다 많이 팔린 제품의 목록 추출
```SQL
USE SalesOrdersSample;

DROP VIEW IF EXISTS AveragePerCategory;
DROP VIEW IF EXISTS TotalPerProduct;

CREATE VIEW TotalPerProduct AS -- 4분기 제품 당 총 판매액
SELECT P2.CategoryID, 
       SUM(OD2.QuotedPrice * OD2.QuantityOrdered) 
       AS SumCategory 
      FROM Products AS P2 
      INNER JOIN Order_Details AS OD2 
        ON P2.ProductNumber = OD2.ProductNumber 
      INNER JOIN Orders AS O2
        ON O2.OrderNumber = OD2.OrderNumber
      WHERE O2.OrderDate BETWEEN '2015-10-01' AND '2015-12-31'
      GROUP BY P2.CategoryID, P2.ProductNumber;
	  
CREATE VIEW AveragePerCategory AS -- 4분기 CategoryID 당 평균 판매액
SELECT t.CategoryID, AVG(t.SumCategory) AS AverageOfCategory 
   FROM TotalPerProduct AS t
   GROUP BY t.CategoryID;

SELECT P.CategoryID, C.CategoryDescription, P.ProductName, 
  SUM(OD.QuotedPrice * OD.QuantityOrdered) AS TotalSales
FROM Products AS P 
  INNER JOIN Order_Details AS OD 
     ON P.ProductNumber=OD.ProductNumber
  INNER JOIN Categories AS C
     ON C.CategoryID = P.CategoryID
  INNER JOIN Orders AS O
     ON O.OrderNumber = OD.OrderNumber
WHERE O.OrderDate BETWEEN '2015-10-01' AND '2015-12-31'
GROUP BY P.CategoryID, C.CategoryDescription, P.ProductName
HAVING SUM(OD.QuotedPrice * OD.QuantityOrdered) > -- 제품 별 총 판매액이 카테고리 별 평균(카테고리의 모든 제품의 합/카테고리 내 제품 수)보다 크다는 조건
	(SELECT a.AverageOfCategory 
	FROM AveragePerCategory AS a 
	WHERE a.CategoryID = P.CategoryID)
ORDER BY CategoryDescription, ProductName;

DROP VIEW IF EXISTS AveragePerCategory;
DROP VIEW IF EXISTS TotalPerProduct;
```
| CategoryID | CategoryDescription | ProductName | TotalSales | 
| -: | - | - | -: | 
| 1 | Accessories | Cycle-Doc Pro Repair Stand | 32595.76 | 
| 1 | Accessories | Dog Ear Aero-Flow Floor Pump | 15539.15 | 
| 1 | Accessories | Glide-O-Matic Cycling Helmet | 23640.00 | 
| 1 | Accessories | King Cobra Helmet | 27847.26 | 
| 1 | Accessories | Viscount CardioSport Sport Watch | 16469.79 | 
| 2 | Bikes | GT RTS-2 Mountain Bike | 527703.00 | 
| 2 | Bikes | Trek 9000 Mountain Bike | 954516.00 | 
| 5 | Car racks | Ultimate Export 2G Car Rack | 31014.00 | 
| 3 | Clothing | StaDry Cycling Pants | 8641.56 | 
| 4 | Components | AeroFlo ATB Wheels | 37709.28 | 
| 4 | Components | Cosmic Elite Road Warrior Wheels | 32064.45 | 
| 4 | Components | Eagle SA-120 Clipless Pedals | 17003.85 | 
| 7 | Skateboards | Viscount Skateboard | 196964.30 | 
| 6 | Tires | Ultra-2K Competition Tire | 5216.28 | 


----

```SQL
USE SalesOrdersSample;

DROP VIEW IF EXISTS CatProdData;
DROP VIEW IF EXISTS AveragePerCategory;
DROP VIEW IF EXISTS TotalPerProductNumber;

CREATE VIEW TotalPerProductNumber AS -- 4분기 제품 당 총 판매액
SELECT 
  P2.CategoryID, P2.ProductNumber,
  SUM(OD2.QuotedPrice * OD2.QuantityOrdered) AS SumCategory 
FROM Products AS P2 
INNER JOIN Order_Details AS OD2 
  ON P2.ProductNumber = OD2.ProductNumber 
INNER JOIN Orders AS O2
  ON O2.OrderNumber = OD2.OrderNumber
WHERE O2.OrderDate BETWEEN DATE '2015-10-01' AND DATE '2015-12-31'
GROUP BY P2.CategoryID, P2.ProductNumber;

CREATE VIEW AveragePerCategory AS -- 4분기 CategoryID 당 평균 판매액
SELECT t.CategoryID, AVG(t.SumCategory) AS AverageOfCategory 
FROM TotalPerProductNumber AS t
GROUP BY t.CategoryID;

CREATE VIEW CatProdData AS -- 4분기 전체 제품 판매 테이블
SELECT C.CategoryID, C.CategoryDescription, P.ProductName, OD.QuotedPrice, OD.QuantityOrdered
FROM Products AS P 
INNER JOIN Order_Details AS OD 
  ON P.ProductNumber=OD.ProductNumber
INNER JOIN Categories AS C
  ON C.CategoryID = P.CategoryID
INNER JOIN Orders AS O
  ON O.OrderNumber = OD.OrderNumber
WHERE O.OrderDate BETWEEN DATE '2015-10-01' AND DATE '2015-12-31';

-- 이건 뭔지...?
SELECT D.CategoryDescription, D.ProductName, 
  SUM(D.QuotedPrice * D.QuantityOrdered) AS TotalSales
FROM CatProdData AS D
GROUP BY D.CategoryID, D.CategoryDescription, D.ProductName
HAVING SUM(D.QuotedPrice * D.QuantityOrdered) > 
  (
	SELECT a.AverageOfCategory 
	FROM AveragePerCategory AS a 
	WHERE a.CategoryID = D.CategoryID
)
ORDER BY CategoryDescription, ProductName;

-- 위의 쿼리와 동일  
SELECT D.CategoryDescription, D.ProductName,  
  SUM(D.QuotedPrice * D.QuantityOrdered) AS TotalSales, a.AverageOfCategory
FROM CatProdData AS D INNER JOIN AveragePerCategory AS a
ON D.CategoryID = A.CategoryID 
GROUP BY D.CategoryID, D.CategoryDescription, D.ProductName
HAVING SUM(D.QuotedPrice * D.QuantityOrdered) > 
  a.AverageOfCategory -- 제품 별 총 판매액이 카테고리 별 평균(카테고리의 모든 제품의 합/카테고리 내 제품 수)보다 크다는 조건
ORDER BY CategoryDescription, ProductName;

DROP VIEW IF EXISTS CatProdData;
DROP VIEW IF EXISTS AveragePerCategory;
DROP VIEW IF EXISTS TotalPerProductNumber;
```

| CategoryDescription | ProductName | TotalSales | 
| - | - | -: | 
| Bikes | GT RTS-2 Mountain Bike | 527703.00 | 
| Bikes | Trek 9000 Mountain Bike | 954516.00 | 
| Skateboards | Viscount Skateboard | 196964.30 | 

---
| CategoryDescription | ProductName | TotalSales | AverageOfCategory | 
| - | - | -: | -: | 
| Accessories | Cycle-Doc Pro Repair Stand | 32595.76 | 9726.661579 | 
| Accessories | Dog Ear Aero-Flow Floor Pump | 15539.15 | 9726.661579 | 
| Accessories | Glide-O-Matic Cycling Helmet | 23640.00 | 9726.661579 | 
| Accessories | King Cobra Helmet | 27847.26 | 9726.661579 | 
| Accessories | Viscount CardioSport Sport Watch | 16469.79 | 9726.661579 | 
| Bikes | GT RTS-2 Mountain Bike | 527703.00 | 507693.000000 | 
| Bikes | Trek 9000 Mountain Bike | 954516.00 | 507693.000000 | 
| Car racks | Ultimate Export 2G Car Rack | 31014.00 | 30493.125000 | 
| Clothing | StaDry Cycling Pants | 8641.56 | 5330.300000 | 
| Components | AeroFlo ATB Wheels | 37709.28 | 16193.426667 | 
| Components | Cosmic Elite Road Warrior Wheels | 32064.45 | 16193.426667 | 
| Components | Eagle SA-120 Clipless Pedals | 17003.85 | 16193.426667 | 
| Skateboards | Viscount Skateboard | 196964.30 | 104473.920000 | 
| Tires | Ultra-2K Competition Tire | 5216.28 | 3942.233333 | 


### BW34. COUNT 함수 이용법
```sql
SELECT Recipe_Classes.RecipeClassDescription, 
  COUNT(Recipes.RecipeClassID) AS RecipeCount
FROM Recipe_Classes 
  LEFT OUTER JOIN Recipes 
    ON Recipe_Classes.RecipeClassID = Recipes.RecipeClassID
GROUP BY Recipe_Classes.RecipeClassDescription
```
| RecipeClassDescription | RecipeCount | 
| - | -: | 
| Dessert | 2 | 
| Hors d'oeuvres | 2 | 
| Main course | 7 | 
| Salad | 1 | 
| Soup | 0 | 
| Starch | 1 | 
| Vegetable | 2 | 

---
```sql
SELECT Recipe_Classes.RecipeClassDescription, COUNT(Recipes.RecipeClassID) AS RecipeCount
FROM Recipe_Classes 
  LEFT OUTER JOIN Recipes 
    ON Recipe_Classes.RecipeClassID = Recipes.RecipeClassID
GROUP BY Recipe_Classes.RecipeClassDescription
```
| RecipeClassDescription | RecipeCount | 
| - | -: | 
| Dessert | 2 | 
| Hors d'oeuvres | 2 | 
| Main course | 7 | 
| Salad | 1 | 
| Soup | 0 | 
| Starch | 1 | 
| Vegetable | 2 | 

---
```sql
SELECT Recipe_Classes.RecipeClassDescription, 
  (SELECT COUNT(Recipes.RecipeClassID) 
   FROM Recipes
   WHERE Recipes.RecipeClassID = Recipe_Classes.RecipeClassID) 
    AS RecipeCount
FROM Recipe_Classes
```
| RecipeClassDescription | RecipeCount | 
| - | -: | 
| Main course | 7 | 
| Vegetable | 2 | 
| Starch | 1 | 
| Salad | 1 | 
| Hors d'oeuvres | 2 | 
| Dessert | 2 | 
| Soup | 0 | 







