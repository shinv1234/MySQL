# Effective SQL Ch6

### BW40. 서브쿼리의 이용

```sql
SELECT BeefRecipes.RecipeTitle
FROM 
  (SELECT Recipes.RecipeID, Recipes.RecipeTitle
   FROM (Recipes INNER JOIN Recipe_Ingredients
    ON Recipes.RecipeID = Recipe_Ingredients.RecipeID) 
      INNER JOIN Ingredients 
    ON Ingredients.IngredientID = 
      Recipe_Ingredients.IngredientID
   WHERE Ingredients.IngredientName = 'Beef') 
      AS BeefRecipes
  INNER JOIN
  (SELECT Recipe_Ingredients.RecipeID
   FROM Recipe_Ingredients INNER JOIN Ingredients
    ON Ingredients.IngredientID = 
      Recipe_Ingredients.IngredientID
   WHERE Ingredients.IngredientName = 'Garlic') 
      AS GarlicRecipes 
    ON BeefRecipes.RecipeID = GarlicRecipes.RecipeID
```
| RecipeTitle | 
| - | 
| Roast Beef | 
- 테이블 서브쿼리의 이용


알 수 없는 테이블
---
```sql
-- Second example using LIKE to fetch real data
SELECT Customers.CustomerID, Customers.CustFirstName, 
  Customers.CustLastName, Orders.OrderNumber, Orders.OrderDate
FROM Customers
  INNER JOIN Orders
    ON Customers.CustomerID = Orders.CustomerID
WHERE EXISTS 
  (SELECT NULL
   FROM (Orders AS O2
      INNER JOIN Order_Details
        ON O2.OrderNumber = Order_Details.OrderNumber)
      INNER JOIN Products
        ON Products.ProductNumber = Order_Details.ProductNumber 
   WHERE Products.ProductName LIKE '%Skateboard%' 
    AND O2.OrderNumber = Orders.OrderNumber)
AND EXISTS 
  (SELECT NULL
   FROM (Orders AS O3 
      INNER JOIN Order_Details
        ON O3.OrderNumber = Order_Details.OrderNumber)
      INNER JOIN Products
        ON Products.ProductNumber = Order_Details.ProductNumber 
   WHERE Products.ProductName LIKE '%Helmet%'
      AND O3.OrderNumber = Orders.OrderNumber)
```
| CustomerID | CustFirstName | CustLastName | OrderNumber | OrderDate | 
| -: | - | - | -: | - | 
| 1002 | William | Thompson | 696 | 2016-01-15 | 
| 1002 | William | Thompson | 852 | 2016-02-13 | 
| 1003 | Gary | Hallmark | 860 | 2016-02-16 | 
| 1004 | Doug | Steele | 39 | 2015-09-07 | 
| 1004 | Doug | Steele | 59 | 2015-09-09 | 
- EXISTS 조건 + 테이블 서브쿼리




