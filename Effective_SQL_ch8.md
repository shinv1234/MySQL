# Ch8

### 등분위로 로우 순위 매기기

#### products 테이블
```sql
SELECT * FROM Products;
```
| ProductNumber | ProductName | ProductDescription | ProductUPD | RetailPrice | QuantityOnHand | CategoryID | 
| -: | - | - | - | -: | -: | -: | 
| 1 | Trek 9000 Mountain Bike | \n | \n | 1200.00 | 6 | 2 | 
| 2 | Eagle FS-3 Mountain Bike | \n | \n | 1800.00 | 8 | 2 | 
| 3 | Dog Ear Cyclecomputer | \n | \n | 75.00 | 20 | 1 | 
| 4 | Victoria Pro All Weather Tires | \n | \n | 54.95 | 20 | 4 | 
| 5 | Dog Ear Helmet Mount Mirrors | \n | \n | 7.45 | 12 | 1 | 
| 6 | Viscount Skateboard | \n | \n | 635.00 | 5 | 7 | 
| 7 | Viscount C-500 Wireless Bike Computer | \n | \n | 49.00 | 30 | 1 | 

#### categories 테이블

```sql
SELECT * FROM Categories
```
| CategoryID | CategoryDescription | 
| -: | - | 
| 1 | Accessories | 
| 2 | Bikes | 
| 3 | Clothing | 
| 4 | Components | 
| 5 | Car racks | 
| 6 | Tires | 
| 7 | Skateboards | 


#### order_details 테이블

```sql
SELECT * FROM Order_Details
```
| OrderNumber | ProductNumber | QuotedPrice | QuantityOrdered | 
| -: | -: | -: | -: | 
| 1 | 1 | 1200.00 | 2 | 
| 1 | 6 | 635.00 | 3 | 
| 1 | 11 | 1650.00 | 4 | 
| 1 | 16 | 28.00 | 1 | 
| 1 | 21 | 55.00 | 3 | 
| 1 | 26 | 121.25 | 5 | 
| 1 | 40 | 174.60 | 6 | 
| 2 | 27 | 24.00 | 4 | 
| 2 | 40 | 180.00 | 4 | 




#### ProdSale 테이블
```sql
CREATE VIEW ProdSale AS
  (SELECT OD.ProductNumber,
       SUM(OD.QuantityOrdered * OD.QuotedPrice) AS ProductSales
     FROM Order_Details AS OD
     WHERE OD.ProductNumber IN 
       (SELECT P.ProductNumber
        FROM Products AS P INNER JOIN Categories AS C
          ON P.CategoryID = C.CategoryID
        WHERE C.CategoryDescription = 'Accessories')
     GROUP BY OD.ProductNumber);

SELECT * FROM ProdSale;
```
| ProductNumber | ProductSales | 
| -: | -: | 
| 3 | 2238.75 | 
| 5 | 767.73 | 
| 7 | 18046.70 | 
| 8 | 5999.50 | 
| 9 | 12488.85 | 
| 10 | 4219.20 | 
| 16 | 14792.96 | 
| 18 | 27954.43 | 
| 19 | 17720.41 | 


#### RankedCategories 테이블

```sql
CREATE VIEW RankedCategories AS
  (SELECT Categories.CategoryDescription, Products.ProductName, 
          ProdSale.ProductSales, 
          (SELECT Count(ProductNumber)
		   FROM ProdSale AS P2 
           WHERE P2.ProductSales > ProdSale.ProductSales) + 1 AS RankInCategory
   FROM Categories 
	INNER JOIN Products 
     ON Categories.CategoryID = Products.CategoryID
   INNER JOIN ProdSale
     ON ProdSale.ProductNumber = Products.ProductNumber
	 ORDER BY ProductSales Desc);

SELECT * FROM RankedCategories;
```
| CategoryDescription | ProductName | ProductSales | RankInCategory | 
| - | - | -: | -: | 
| Accessories | Cycle-Doc Pro Repair Stand | 62157.04 | 1 | 
| Accessories | King Cobra Helmet | 57572.41 | 2 | 
| Accessories | Glide-O-Matic Cycling Helmet | 56286.25 | 3 | 
| Accessories | Dog Ear Aero-Flow Floor Pump | 36029.40 | 4 | 
| Accessories | Viscount CardioSport Sport Watch | 27954.43 | 5 | 
| Accessories | Pro-Sport 'Dillo Shades | 20336.82 | 6 | 



#### ProdCount 테이블
```sql
CREATE VIEW ProdCount AS
  (SELECT COUNT(ProductNumber) AS NumProducts 
   FROM ProdSale); 

SELECT * FROM ProdCount;
```
| NumProducts | 
| -: | 
| 19 | 


#### 최종 쿼리
```sql
SELECT P1.CategoryDescription, P1.ProductName, 
    P1.ProductSales, P1.RankInCategory, 
    (CASE WHEN RankInCategory <= ROUND(0.2 * NumProducts, 0)
            THEN 'First' 
          WHEN RankInCategory <= ROUND(0.4 * NumProducts,0)
            THEN 'Second' 
          WHEN RankInCategory <= ROUND(0.6 * NumProducts,0)
            THEN 'Third' 
          WHEN RankInCategory <= ROUND(0.8 * NumProducts,0)
            THEN 'Fourth' 
          ELSE 'Fifth' END) AS Quintile
FROM RankedCategories AS P1 
CROSS JOIN ProdCount
ORDER BY P1.ProductSales DESC;
```
| CategoryDescription | ProductName | ProductSales | RankInCategory | Quintile | 
| - | - | -: | -: | - | 
| Accessories | Cycle-Doc Pro Repair Stand | 62157.04 | 1 | First | 
| Accessories | King Cobra Helmet | 57572.41 | 2 | First | 
| Accessories | Glide-O-Matic Cycling Helmet | 56286.25 | 3 | First | 
| Accessories | Dog Ear Aero-Flow Floor Pump | 36029.40 | 4 | First | 
| Accessories | Viscount CardioSport Sport Watch | 27954.43 | 5 | Second | 
| Accessories | Pro-Sport 'Dillo Shades | 20336.82 | 6 | Second | 
| Accessories | Viscount C-500 Wireless Bike Computer | 18046.70 | 7 | Second | 
| Accessories | Viscount Tru-Beat Heart Transmitter | 17720.41 | 8 | Second | 
| Accessories | HP Deluxe Panniers | 15984.54 | 9 | Third | 
| Accessories | ProFormance Knee Pads | 14792.96 | 10 | Third | 
| Accessories | Ultra-Pro Knee Pads | 14581.35 | 11 | Third | 
| Accessories | Nikoma Lok-Tight U-Lock | 12488.85 | 12 | Fourth | 
| Accessories | TransPort Bicycle Rack | 9442.44 | 13 | Fourth | 
| Accessories | True Grip Competition Gloves | 7465.70 | 14 | Fourth | 
| Accessories | Kryptonite Advanced 2000 U-Lock | 5999.50 | 15 | Fourth | 
| Accessories | Viscount Microshell Helmet | 4219.20 | 16 | Fifth | 
| Accessories | Dog Ear Monster Grip Gloves | 2779.50 | 17 | Fifth | 
| Accessories | Dog Ear Cyclecomputer | 2238.75 | 18 | Fifth | 
| Accessories | Dog Ear Helmet Mount Mirrors | 767.73 | 19 | Fifth | 




### 한 테이블에서 각 행과 다른 모든 행들을 쌍으로 만드는 방법

#### CROSS JOIN을 이용한 매칭
```sql
SELECT Teams1.TeamID AS Team1ID, 
       Teams1.TeamName AS Team1Name, 
       Teams2.TeamID AS Team2ID, 
       Teams2.TeamName AS Team2Name
FROM Teams AS Teams1 CROSS JOIN Teams AS Teams2
WHERE Teams2.TeamID > Teams1.TeamID -- 중복 매칭의 제거
ORDER BY Teams1.TeamID, Teams2.TeamID;
```
| Team1ID | Team1Name | Team2ID | Team2Name | 
| -: | - | -: | - | 
| 1 | Marlins | 2 | Sharks | 
| 1 | Marlins | 3 | Terrapins | 
| 1 | Marlins | 4 | Barracudas | 
| 1 | Marlins | 5 | Dolphins | 
| 1 | Marlins | 6 | Orcas | 
| 1 | Marlins | 7 | Manatees | 
| 1 | Marlins | 8 | Swordfish | 
| 1 | Marlins | 9 | Huckleberrys | 
| 1 | Marlins | 10 | MintJuleps | 
| 2 | Sharks | 3 | Terrapins | 
| 2 | Sharks | 4 | Barracudas | 
| 2 | Sharks | 5 | Dolphins | 
| 2 | Sharks | 6 | Orcas | 
| 2 | Sharks | 7 | Manatees | 
| 2 | Sharks | 8 | Swordfish | 
| 2 | Sharks | 9 | Huckleberrys | 
| 2 | Sharks | 10 | MintJuleps | 
| 3 | Terrapins | 4 | Barracudas | 
| 3 | Terrapins | 5 | Dolphins | 
| 3 | Terrapins | 6 | Orcas | 
| 3 | Terrapins | 7 | Manatees | 
| 3 | Terrapins | 8 | Swordfish | 
| 3 | Terrapins | 9 | Huckleberrys | 
| 3 | Terrapins | 10 | MintJuleps | 
- WHERE문 제거 시 중복매칭 발생 (Ex> 1: 1 2 3, 2: 1, 2, 3, 3: 1, 2, 3)

#### INNER JOIN을 이용한 매칭
```sql
SELECT Teams1.TeamID AS Team1ID, 
       Teams1.TeamName AS Team1Name, 
       Teams2.TeamID AS Team2ID, 
       Teams2.TeamName AS Team2Name
FROM Teams AS Teams1 INNER JOIN Teams AS Teams2
   ON Teams2.TeamID > Teams1.TeamID -- 중복 매칭의 제거
ORDER BY Teams1.TeamID, Teams2.TeamID;
```
| Team1ID | Team1Name | Team2ID | Team2Name | 
| -: | - | -: | - | 
| 1 | Marlins | 2 | Sharks | 
| 1 | Marlins | 3 | Terrapins | 
| 1 | Marlins | 4 | Barracudas | 
| 1 | Marlins | 5 | Dolphins | 
| 1 | Marlins | 6 | Orcas | 
| 1 | Marlins | 7 | Manatees | 
| 1 | Marlins | 8 | Swordfish | 
| 1 | Marlins | 9 | Huckleberrys | 
| 1 | Marlins | 10 | MintJuleps | 
| 2 | Sharks | 3 | Terrapins | 
| 2 | Sharks | 4 | Barracudas | 
| 2 | Sharks | 5 | Dolphins | 
| 2 | Sharks | 6 | Orcas | 
| 2 | Sharks | 7 | Manatees | 
| 2 | Sharks | 8 | Swordfish | 
| 2 | Sharks | 9 | Huckleberrys | 
| 2 | Sharks | 10 | MintJuleps | 
| 3 | Terrapins | 4 | Barracudas | 
| 3 | Terrapins | 5 | Dolphins | 
| 3 | Terrapins | 6 | Orcas | 
| 3 | Terrapins | 7 | Manatees | 
| 3 | Terrapins | 8 | Swordfish | 
| 3 | Terrapins | 9 | Huckleberrys | 
| 3 | Terrapins | 10 | MintJuleps | 
- ON문 제거 시 중복매칭 발생 (Ex> 1: 1 2 3, 2: 1, 2, 3, 3: 1, 2, 3)

#### 한 번에 제품을 3개 고르는 모든 조합 찾기

```sql
SELECT Prod1.ProductNumber AS P1Num, Prod1.ProductName AS P1Name,
    Prod2.ProductNumber AS P2Num, Prod2.ProductName AS P2Name,
    Prod3.ProductNumber AS P3Num, Prod3.ProductName AS P3Name
FROM Products AS Prod1 CROSS JOIN Products AS Prod2 
   CROSS JOIN Products AS Prod3
WHERE Prod1.ProductNumber < Prod2.ProductNumber AND
    Prod2.ProductNumber < Prod3.ProductNumber
```
| P1Num | P1Name | P2Num | P2Name | P3Num | P3Name | 
| -: | - | -: | - | -: | - | 
| 1 | Trek 9000 Mountain Bike | 2 | Eagle FS-3 Mountain Bike | 3 | Dog Ear Cyclecomputer | 
| 1 | Trek 9000 Mountain Bike | 2 | Eagle FS-3 Mountain Bike | 4 | Victoria Pro All Weather Tires | 
| 1 | Trek 9000 Mountain Bike | 2 | Eagle FS-3 Mountain Bike | 5 | Dog Ear Helmet Mount Mirrors | 
| 1 | Trek 9000 Mountain Bike | 2 | Eagle FS-3 Mountain Bike | 6 | Viscount Skateboard | 
| 1 | Trek 9000 Mountain Bike | 2 | Eagle FS-3 Mountain Bike | 7 | Viscount C-500 Wireless Bike Computer | 

- WHERE문을 주의!







