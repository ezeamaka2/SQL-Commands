# SQL-Commands
SELECT DISTINCT TOP 3
col 1
SUM(column)FROM table_name
WHERE condition
GROUP BY col 1
HAVING SUM(column) > condition
ORDER BY column DESC/ASC

Data Definition Command
## Adding a new column to an existing table
ALTER TABLE Table_name
ADD email VARCHAR(50) NOT NULL 

## Deleting a particular column from a table
ALTER TABLE Table_name
DROP COLUMN column_name

#Deleting a Table
DROP table_name

UPDATING A ROW
UPDATE table_name
SET score = 343
WHERE id=6 (id = the id of the row you want to update)


UPDATE customers
SET score = 0
WHERE score IS NULL

DELETE FROM customers
WHERE id>5

SELECT *
FROM customers
WHERE id>5

INSERT INTO customers(id, first_name, country,score)
VALUES (10,'Sahra', NULL, NULL)

To delete everything from a Table
DELETE FROM table_name
TRUNCATE TABLE table_name - faster for deleting everything from a large table

JOINING DATA
_INNER JOIN (Joining only matching data)
SELECT *
FROM table_a
INNER JOIN table_b
ON <conditions (example table_a.key = table_b.key)>

_LEFT JOIN
This join returns all the rows from the left and only the matching data from the right
SELECT *
FROM table_a
LEFT JOIN table_b
ON <conditions (example table_a.key = table_b.key)>


SELECT 
	O.OrderID,
	O.Sales,
	C.FirstName,
	C.LastName,
	P.Product,
	P.Price,
	E.FirstName,
	E.LastName
FROM Sales.Orders AS O
LEFT JOIN Sales.Customers AS C
ON O.CustomerID = C.CustomerID
LEFT JOIN Sales.Products AS P
ON O.ProductID = P.ProductID
LEFT JOIN Sales.Employees AS E
ON O.SalesPersonID = E.EmployeeID


SELECT
	Product,
	Price,
	AveragePrice
FROM(
	SELECT
	ProductID,
	Product,
	Price,
	AVG(Price) OVER() AS AveragePrice
FROM Sales.Products
)t
WHERE Price > AveragePrice

SELECT
	*,
	ROW_NUMBER() OVER(ORDER BY RankCust DESC)
FROM(
	SELECT
	CustomerID,
	SUM(Sales) As RankCust
FROM Sales.Orders
GROUP BY CustomerID
)t


SELECT
	ProductID,
	Product,
	Price,
	(SELECT COUNT(OrderID) FROM Sales.Orders) AS TotalOrders
FROM Sales.Products


SELECT
	*
FROM Sales.Customers l
LEFT JOIN
(SELECT
	CustomerID,
	COUNT(CustomerID) AS toalOrder
FROM Sales.Orders r
GROUP BY CustomerID)t
ON l.CustomerID = t.CustomerID


SELECT
	*
FROM Sales.Orders
WHERE CustomerID IN (
	SELECT
	CustomerID
FROM Sales.Customers
WHERE Country = 'Germany'
)


SELECT
	*
FROM Sales.Employees
WHERE Gender = 'F' AND Salary >ALL (SELECT
	Salary
FROM Sales.Employees
WHERE Gender = 'M')


Common Table Expression -CTE
--CTE Query
--Get the total sales by customers
WITH CTE_TotalSales AS (
	SELECT
	CustomerID,
	SUM(Sales) AS TotalSales
FROM Sales.Orders
GROUP BY CustomerID
),
--Get the last order date
--Second CTE
CTE_LastOrderDate AS (
	SELECT
		CustomerID,
		MAX(OrderDate) AS LastOrder
	FROM Sales.Orders
	GROUP BY CustomerID
),
-- Rank customers based on totalSales per customers - Nested CTE example
CTE_Rank_Customers AS (
	SELECT
		CustomerID,
		TotalSales,
		RANK() OVER(ORDER BY TotalSales DESC) AS CustomerRank
	FROM CTE_TotalSales
),
--Segment the customers based on totalSales
CTE_Customer_Segment AS (
	SELECT
		CustomerID,
		CASE 
			WHEN TotalSales > 100 THEN 'High'
			WHEN TotalSales > 50 THEN 'Medium'
			ELSE 'Low'
		END AS CustomerSegment
	FROM CTE_TotalSales
)

-- MAIN Query
SELECT
	c.CustomerID,
	c.FirstName,
	c.LastName,
	COALESCE(cts.TotalSales, 0) SalesByCust,
	(lod.LastOrder) As LastOrderDate,
	crnk.CustomerRank,
	ccs.CustomerSegment
FROM Sales.Customers c
LEFT JOIN CTE_TotalSales cts
ON c.CustomerID = cts.CustomerID
LEFT JOIN CTE_LastOrderDate lod
ON c.CustomerID = lod.CustomerID
LEFT JOIN CTE_Rank_Customers crnk
ON c.CustomerID = crnk.CustomerID
LEFT JOIN CTE_Customer_Segment ccs
ON c.CustomerID = ccs.CustomerID


VIEW
CREATE VIEW Sales.V_Order_Details_EU AS (
SELECT
	o.OrderID,
	OrderDate,
	p.Product,
	p.Category,
	CONCAT(c.FirstName, ' ' , c.LastName) CustomerName,
	c.Country CustomerCountry,
	COALESCE(e.FirstName, ' ') + ' ' + COALESCE(e.LastName, ' ') SalesName,
	e.Department,
	o.Sales,
	o.Quantity
FROM Sales.Orders o
LEFT JOIN Sales.Products p
ON o.ProductID = p.ProductID
LEFT JOIN Sales.Customers c
ON o.CustomerID = c.CustomerID
LEFT JOIN Sales.Employees e
ON o.SalesPersonID = e.EmployeeID
WHERE c.Country <> 'USA'
)


SELECT
	*
FROM Sales.V_Order_Details_EU



SELECT
	*
	INTO #Orders
FROM Sales.Orders

DELETE FROM #Orders
WHERE OrderStatus = 'Delivered'

SELECT
	*
	INTO Sales.OrderTest
FROM #Orders

SELECT * FROM Sales.OrderTest


STORED PROCEDURE
--Creating a stored procedure
ALTER PROCEDURE USACustomerSummary AS
BEGIN
	SELECT
		COUNT(*) TotalCustomer,
		AVG(score) AvgScore
	FROM Sales.Customers
	WHERE Country = 'USA'
END

EXEC USACustomerSummary

-- Passing paramter into a stored procedureb 

ALTER PROCEDURE USACustomerSummary @Country NVARCHAR(50) AS
BEGIN
	SELECT
		COUNT(*) TotalCustomer,
		AVG(score) AvgScore
	FROM Sales.Customers
	WHERE Country = @Country
END

-- Procedure with default value
CREATE PROCEDURE USACustomerSummary @Country NVARCHAR(50) = 'USA' AS
BEGIN
	SELECT
		COUNT(*) TotalCustomer,
		AVG(score) AvgScore
	FROM Sales.Customers
	WHERE Country = @Country;


	-- Multiple SQL Query in a stored procedure
-- Find the toal number of orders and total sales

SELECT
	COUNT(OrderID) TotalOrders,
	SUM(Sales) TotalSales
FROM Sales.Orders o
LEFT JOIN Sales.Customers c
ON c.CustomerID = o.CustomerID
WHERE c.Country = @Country;

END

EXEC USACustomerSummary 'Germany'
EXEC USACustomerSummary @Country = 'USA'

-- To delete procedure
DROP PROCEDURE USACustomerSummary


-- Working with variable in a stored procedure
-- Procedure with default value
ALTER PROCEDURE USACustomerSummary @Country NVARCHAR(50) = 'USA' AS
BEGIN

DECLARE @TotalCustomer INT, @AvgScore FLOAT;

-- Prepare and clean up data

IF EXISTS (SELECT 1 FROM Sales.Customers WHERE Country = @Country AND Score IS NULL)
BEGIN
	PRINT('Updating null')
	UPDATE Sales.Customers
	SET Score = 0
	WHERE Country = @Country AND Score IS NULL;
END

ELSE
BEGIN
	PRINT('No null value was found')
END;

--Generate report
	SELECT
		@TotalCustomer = COUNT(*),
		@AvgScore = AVG(score)
	FROM Sales.Customers
	WHERE Country = @Country;


PRINT 'Total Customer from ' + @Country + ' : ' + CAST(@TotalCustomer AS NVARCHAR)
PRINT 'Average Score from ' + @Country + ' : ' + CAST(@AvgScore AS NVARCHAR)

SELECT
	COUNT(OrderID) TotalOrders,
	SUM(Sales) TotalSales
FROM Sales.Orders o
LEFT JOIN Sales.Customers c
ON c.CustomerID = o.CustomerID
WHERE c.Country = @Country;

END

SELECT * FROM Sales.Customers


--TRIGGERS
CREATE TABLE Sales.EmployeeLog (
	LogID INT IDENTITY(1,1) PRIMARY KEY,
	EmployeeID INT,
	LogMessage VARCHAR(255),
	LogDate DATE
)

SELECT * FROM Sales.EmployeeLog

--CREATEING TRIGGER
CREATE TRIGGER tri_AfterInsertEmployee ON Sales.Employees
AFTER INSERT
AS
BEGIN
	INSERT INTO Sales.EmployeeLog(EmployeeID, LogMessage, LogDate)
	SELECT
		EmployeeID,
		'New employee has been added: ' + CAST(EmployeeID AS VARCHAR),
		GETDATE()
	FROM INSERTED
END


--TRIGGERING THE TRIGGER By Inserting a new data 
INSERT INTO Sales.Employees
VALUES
(7, 'Maria', 'Doe', 'HR', '1900-01-12', 'F', 80000, 3)

