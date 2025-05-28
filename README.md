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
ALTER TABLLE Table_name
ADD email VARCHAR(50) NOT NULL 

## Deleting a particular column from a table
ALTER TABLLE Table_name
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
