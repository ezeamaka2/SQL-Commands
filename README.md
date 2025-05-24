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
