# Write Advanced Transact-SQL Queries

https://learn.microsoft.com/en-us/training/paths/write-advanced-transact-sql-queries/

T-SQL is the extension of SQL used in Microsoft SQL Server relational database engine and closely related Microsoft data platforms.

While SQL is the base general language, T-SQL is Microsoft's dialect of SQL, with extra syntax, functions, procedural features, error handling, variabls, stored procedures, and SQL-Server specific capabilities.

---

## Create and query tables

### Create tables

To create a table we need to do the following:

1. Point to our database, e.g., OnlineShop database:

```sql
USE OnlineShop;
```

2. Use `CREATE TABLE`, e.g., Product table:

```sql
CREATE TABLE Products  
(
    ProductID int PRIMARY KEY NOT NULL,  
    ProductName varchar(50) NOT NULL,  
    ProductDescription varchar(max) NOT NULL
);
```

> Creating a table has 3 requirements: table name, column names, column data types. Note: you also need valid permissions.

---

### Insert and read data

After creating a table you populate with `INSERT`, e.g., into the Product table:

```sql
INSERT Products (ProductID, ProductName, ProductDescription)
    VALUES (1, 'Example Product Name', 'Example Product Description');
```
After populating, you may read using `SELECT`, e.g., from the Product table:

```sql
SELECT ProductName ProductDescription
    FROM Products;
```

---

## Create and query views

Views: 
- Saved queries that can be used in the same way as a table but are not persistently stored - unpacked at runtime. 
- Protects from accidental changes to underlying data in application. 
- Can be considered an API (application programming interface) acting as an abstraction layer betweent he applciation and the underlying tables.

---

Use `CREATE VIEW` to name and store a single `SELECT` statement as a view:

```sql
CREATE VIEW ViewName 
AS 
SELECT ...
FROM ...
```

Use `CREATE OR ALTER` if you want to create the view if it doesn't exist or alter the view if it already eists (allowing you to actively make changes to the original query to update the view):

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
- `ENCRYPTION` - obscures the view's deifnition in metadata that prevents someone inspecting the databas3 from retrieving the original.
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
- table only visible to you and is dropped automatically when session ends.
- multiple users can create local temp tables with the same name wiht no effect on each other.

> A stored procedure creates its own scope, so a local temp table created inside of a stored procedure is dropped automatically when the stored procedure ends.

You create a local temporary table the same way in creating a regular table but add a # prior to the table name signifying its status as a local temporary table:

```sql
CREATE TABLE #Products (
    ProductID INT PRIMARY KEY,
    ProductName varchar(50),
    ...
);
```

---

### Create global temporary table

Global temporary tables are shared so it must ahve a unique. name unlike a local temp table. Global tables are dropped automatically when:
- the session that created it ends, and
- all tasks referencing it across all sessions have also ended.

> Between SQL Server (installed software / self-managed) and Azure SQL Database (Microsoft managed Azure service) global temp tables act slightly different.
> - In SQL Server global temp tables are accessible across all sessions on the instance.
> - In Azure SQL Database, the global temp tables are scoped to database level with other databases on the same logical server not being able to access them.

You create a global temp table in the same way but use ## instead of a single #:

```sql
CREATE TABLE ##Products (
    ProductID INT PRIMARY KEY,
    ProductName varchar(50),
    ...
);
```

---

### Insert and read from temp table:

Follows same approach as regular table, ensuring to prefix table name with # or ##.

```sql
SELECT #<TableName> (<ColumnNames>)
    VALUES (<InsertedValues>)
```

```sql
SELECT *
FROM #<TableName>
```

---

## Use common table expressions

CTEs define subqueries at the start of a query to be referenced and used throughout the outer query.

They are named expressions defined in a query whos life is limited in scope to the execution of the outer query so when the outer query ends, so does the CTEs lifetime.

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

- Unless columns are already distinct, in cases of joins, a CTE may require aliases to make them unique.
- Separate queries with `;` is the CTE is following another statement in the same batch.
- Unlike derived tables, CTEs can be referenced multiple times in one query, and multiple queries can be defined using one `WIH` clause.
- CTEs support recursion, in whih the epression is defined with a refernce to itself.

## Write queries that use derived tables

Derived tables, like CTEs, allow fo more modular statements that break down complex queries into manageable parts.

Derived tables can provide qorkarounds for some retrications imposed by logical order of query processing, such as use of column aliases.

Like subqueries, derived table can be created in the `FROM` clause of an outer `SELECT` statement.





