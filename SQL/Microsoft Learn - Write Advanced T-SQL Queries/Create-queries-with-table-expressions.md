# Create tables, views, and temporary objects

https://learn.microsoft.com/en-gb/training/modules/create-tables-views-temporary-objects/1-introduction

Transact-SQL (T-SQL): Microsoft SQL Server (relational database engine) and other Microsoft data platforms' extension of SQL.

Has extra syntax and processes other SQL forms sometimes dont have.

Learn:

- CREATE TABLE, INSERT and SELECT for normal tables and, local and global temp tables.

- Create and use VIEWS, CTEs and Derived Tables.

---

## Create and query tables

### `CREATE TABLE`

First point to database, e.g., OnlineShop database:

```sql
USE OnlineShop;
```

Use `CREATE TABLE`, e.g., Product table:

```sql
CREATE TABLE Products  
(
    ProductID int PRIMARY KEY NOT NULL,  
    ProductName varchar(50) NOT NULL,  
    ProductDescription varchar(max) NOT NULL
);
```

> 3 requirements: table name, column names, column data types. Additionally require create permissions.

---

### `INSERT` and `SELECT` data

Populate created table with `INSERT`:

```sql
INSERT Products (ProductID, ProductName, ProductDescription)
    VALUES (1, 'Example Product Name', 'Example Product Description');
```

Read populated table using `SELECT`:

```sql
SELECT ProductName ProductDescription
    FROM Products;
```

---

## Create and query `VIEWS`

Saved queries - unpacked at runtime - not persistentyly stored.
- Protect from accidental changes to underlying data in application.
- Act as abstraction layer between application and undelrying tables - like an API.

---

Use `CREATE VIEW` to name and store a single `SELECT` statement as a view:

```sql
CREATE VIEW ViewName 
AS 
SELECT ...
FROM ...
```

Use `CREATE OR ALTER` to create new or edit existing view (allows live edits to views without `DROP`):

```sql
CREATE OR ALTER VIEW ViewName 
AS 
SELECT ...
FROM ...
```

You can also explcitely define the view's column names:
```sql
CREATE VIEW ProductView
    (ID, Name, Price)
AS
SELECT
    ProductId, ProductName, ProductPrice
FROM Products;
```

You can also use `WITH` to apply view attribute clauses like `SCHEMABINDING`, `ENCRYPTION` and `VIEW_METADATA`:
```sql
CREATE VIEW ProductView
WITH SCHEMABINDING, ENCRYPTION, VIEW_METADATA
AS
SELECT ...
FROM ...
```
- `SCHEMABINING` - prevents changes to the underlying objects that would break the view, also required for indexing.
- `ENCRYPTION` - obscures the view's deifnition in metadata that prevents someone inspecting the database from retrieving the original.
- `VIEW_METADATA` - alters metadata presented to clint APIs so that the view's metadata is returned rather than the underlying base table metadata.

You can also use `WITH CHECK OPTION` to modify data through this view that would no longer be visible through the view, e.g., cannot insert or update price less than 100 as it would no longer satisfy `Price >= 100`:

```sql
CREATE VIEW ExpensiveProducts
AS
SELECT *
FROM Products
WHERE Price >= 100
WITH CHECK OPTION;
```

>`ORDER BY` is not permitted inside of a view's deifnition unless the view uses `TOP, OFFSET/FETCH, or FOR XML` - this is because the view represents a results set not a guarunteed presentation order.

You would normally put the ordering in querying the view:
```sql
SELECT *
FROM ViewName
ORDER BY Price
```
However with `TOP`, `OFFSET/FETCH`, or `FOR XML` order is meaningful when determining which rows belong to the result set:

- Which n rows should `TOP n` select:

```sql
CREATE VIEW <ViewName>
AS
SELECT TOP n
    <ColumnNames>
FROM <TableName>
ORDER BY <ColumnName>
```

- Which n rows should not be returned when rows are offset:

```sql
CREATE VIEW <ViewName>
AS
SELECT 
    <ColumnNames>
FROM <TableName>
ORDER BY <ColumnName>
OFFSET n ROWS
```

- Which n rows after the offset should be returned:

```sql
CREATE VIEW <ViewName>
AS
SELECT 
    <ColumnNames>
FROM <TableName>
OFFSET n ROWS
FETCH NEXT n ROWS ONLY
```

- When producing XML which is a serialised output, meaning a docuement where the sequence of elements can matter:

```sql
CREATE VIEW <ViewName>
AS
SELECT 
    <ColumnNames>
FROM <TableName>
ORDER BY <ColumnName>
FOR XML AUTO
```

## Use temporary tables

There are two types of temporary tables:
- local temporary tables
- global temporary tables

---

### Create local temporary tables:

Local temp tables create tables scoped to the current session:
- only visible in your user session and dropped when ended.
- no naming comflict across user sessions - local temp tables can have same name.

> A stored procedure creates its own scope, so a local temp table created inside of a stored procedure is dropped automatically when the stored procedure ends.

Same as normal table using `CREATE TABLE` but use # prior to table name:

```sql
CREATE TABLE #Products (
    ProductID INT PRIMARY KEY,
    ProductName varchar(50),
    ...
);
```

---

### Create global temporary table

Global temporary tables are shared so must have unique names. 

Global tables are dropped when:
- the session that created it ends, and
- all tasks referencing it across all sessions have also ended.

> Microsoft SQL Server (installed self-managed software), global temp tables are scoped across all sessions on instance.

> Azure SQL DB (Microsoft managed Azure service), global temp tables are scoped at database level rather than server level.

Use ## instead of a single #:

```sql
CREATE TABLE ##Products (
    ProductID INT PRIMARY KEY,
    ProductName varchar(50),
    ...
);
```

---

### `INSERT` and `SELECT` from temp table:

Follows same approach as regular table, ensuring to prefix table name with # or ##.

```sql
INSERT #<TableName> (<ColumnNames>)
    VALUES (<InsertedValues>)
```

```sql
SELECT *
FROM #<TableName>
```

---

## Use `Common Table Expressions`:

Named expressions - subqueries defined at start - referenced and used throughout outer query - life limited to scope of outer query.

---

### Write queries with CTEs to retrieve results:

Define CTEs using `WITH`, placing subquery inside brackets. The resulting CTE can be referened in the outer query:

```sql
WITH <CTE_name>
AS (<CTE_definition>)

SELECT ...
FROM <CTE_name>
```

When writing CTEs consider the following:

- Resulting aliases/column names must be unique.
- Separate outer queries with ;
- Multiple CTEs can be defined in a single `WITH` clause.
- CTEs support recursion - expression is defined with reference to itself.

## Write queries that use Derived Tables

Allow for modular problem solving - like CTEs.

Provide workaround to logical query processing - e.g., column aliases.

Can be created in `FROM` clause - like subqueries.

No privalages required as not stored in database - other than select.

Life limited to outer query.

Does not necessarily impact performance compared to the same query expressed differently.

---

### Derive results using `Derived Tables`:

Write inner query in parentheses followed by `AS` clause and a name:

```sql
SELECT <outer query_column_list>
FROM (SELECT <inner_query_column_list>
        FROM <table_source>) AS <derived_table_name>
```

Outer query operates on the derived table results.

### Pass argument to `Derived Tables`:

Accepts arguments passed from a calling routine (e.g., T-SQL batch, function, or stored procedure).

Write using local variables as placeholders that are replaced at runtime by values from batch or stored procedure.

Allows flexibility in code and qeury use.

Batch below declares local variables (prefixed with `@`) then assings a value to it:

```sql
DECLARE @emp_id INT = 9; --declare and assign the variable

SELECT
    orderyear,
    COUNT(DISTINCT custid) AS cust_count
FROM (
    SELECT YEAR(orderdate) AS orderyear,
        custid
    FROM Sales.Orders
    WHERE empid = @emp_id --use variable to pass a value to the derived table query
) AS derived_year
GROUP BY orderyear;

GO
```

When using derived tables consider the following:

- Nested `SELECTs` must have assigned alises that the outer query may use in the same way joined tables are aliased.

- Best practice to assign each referenced derived column a unique alias either in line or externally.

- Dervied tables' `SELECT` must not have `ORDER BY` unless including `TOP`, `OFFSET/FETCH`, or `FOR XML` - results are sorted in outer query.

- Derived tables can accept local variables or paramters in stored procedures.

- Nesting of derived tables is possible but not recommended due to complexity.

- Derived tables should not be referenced multiple times in the outer query as it will need to be defined every time.

## Exercise: Create queries with table expressions

https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/06-use-table-expressions.html

Challenge 1: Create view of canadian customer only

```sql

CREATE VIEW SalesLT.vAddressCA
AS
SELECT
    AddressLine1,
    City,
    StateProvince,
    CountryRegion
FROM SalesLT.Address
WHERE CountryRegion = 'Canada';

SELECT *
FROM SalesLT.vAddressCA;
```

Challenge 2: Write qury to classify by weight, then use as derived table

```sql
SELECT 
    ProductId, 
    Name, 
    Weight,
    ListPrice,
    CASE WHEN Weight > 1000 THEN N'Heavy' ELSE N'Normal' END AS WeightType
FROM SalesLT.Product;

SELECT
    dt.ProductId,
    dt.Name,
    dt.wWight,
    dt.ListPrice,
    dt.WeightType
FROM (
    SELECT
        ProductId, 
        Name, 
        Weight,
        ListPrice,
        CASE WHEN Weight > 1000 THEN N'Heavy' ELSE N'Normal' END AS WeightType
    FROM SalesLT.Product
) AS dt
WHERE dt.WeightType = N'Heavy';

```
> The `N'` prior to a string means the string is UNICODE, while normal ' ' means it is a VARCHAR sring. Unicode is important when text contains characters ouside normal character set -  it is a safe habit to have e.g., 

```sql
N'你好'
N'مرحبا'
N'José'
```



