# Combine query results with set operators

https://learn.microsoft.com/en-gb/training/modules/combine-query-results-set-operators/

MS SQL Server has set operators that for operations using sets that result from two or more queries:
- `UNION`: combine rows from two result sets.
- `INTERSECT` and `EXCEPT`: compare results of two result sets.
- `APPLY`: use rsult of one query to collect output of second query.

> `ORDER BY` can be used to order the final result not within the input queries.

---

## Using `UNION`:

Allows you to combine result sets into a single result set:
- `UNION ALL` combined result DO NOT include duplicates.
- `UNION` combined reults DO include duplicates.

> `NULL` is treated as equivalent between sets.

Rules:
- number and order of columns must be the same in all queries.
- date types must be compatable.

> `JOIN` aggregates and compares horizontally while `UNION` concatenates vertically, appending all results from first set to second.

```sql
SELECT FirstName, SecondName
FROM Sales.Customers
WHERE CustomerId BETWEEN 1 AND 9

UNION

SELECT FirstName, SecondName
FROM Sales.Customers
WHERE CustomerId BETWEEN 21 and 29

ORDER BY FirstName
```

> Like all T-SQL queries, no sort order is guarunteed unless specified. Use a `GROUP BY` after the second query.

> Both `UNION` and `UNION ALL` must have same number and types of columns to allow the concatenation.

---

## Use `INTERSECT` and `EXCEPT` operators:

Compare results that are in common (`INTERSECT`) or rows that are in one set but not the other (`EXCEPT`)

On a venn diagram, `INTERSECT` is the section of overlap. Find colours that appear in both sets:

```sql
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 500 and 750

INTERSECT

SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 751 and 1000;
```

On a venn diagram `EXCEPT` returns distinct rows in the first set but not the second:

```sql
SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 500 and 750

EXCEPT

SELECT color FROM SalesLT.Product
WHERE ProductID BETWEEN 751 and 1000;
```

> Results from `EXCEPT` are different depending on which query is first.

---

## Use the `APPLY` operator:

`APPLY` applies table expressions from one set on each row in another set - enables queries that evaludate rows in one input set against the expression that defines the second input set.

> Actually a table operators, not a set operator, acting more like a JOIN rather than a set operator on two compartible query result sets.

Two types of `APPLY`:
- CROSS APPLY
- OUTER APPLY

```sql
SELECT <column_list>
FROM left_table_source { CROSS | OUTER } APPLY right_table_source
```

> `APPLY` is generally used since it looks more natural but is often not always needed and can be equally represneted using joins, window functions and CTEs.

### Use `CROSS APPLY`:

`CROSS APPLY` applies right table source to each row in the left table source - only rows that are in both tables are returned.

```sql
SELECT *
FROM Customers c
CROSS APPLY (
    SELECT *
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
    LIMIT 1
) x;
```

> Most `INNER JOIN` statements can be rewritten as `CROSS APPLY` statements, although more complicated CTE + JOIN statements can be simplified through CROSS APPLY.

```sql
WITH RankedOrders AS (
    SELECT
        OrderID,
        CustomerID,
        OrderDate,
        ROW_NUMBER() OVER (
            PARTITION BY CustomerID
            ORDER BY OrderDate DESC
        ) AS rn
    FROM Orders
)
SELECT
    c.CustomerID,
    o.OrderID,
    o.OrderDate
FROM Customers c
INNER JOIN RankedOrders o
    ON c.CustomerID = o.CustomerID
WHERE o.rn = 1;
```

> T-SQL `CROSS APPLY ... <alias>` MySQL equivalent is `JOIN LATERAL ... <alias> ON TRUE`

```sql
SELECT *
FROM Customers c
JOIN LATERAL (
    SELECT *
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
    LIMIT 1
) x ON TRUE;
```

### Use `OUTER APPLY`:

`OUTER APPLY` preserves every row from left table source, regardless of returned rows from right table epression.

If `CROSS APPLY` acts like `INNER JOIN` then `OUTER APPLY` acts like `LEFT JOIN` - where unmatched rows appear in results with NULL values.

```sql
SELECT
    c.CustomerID,
    x.OrderID,
    x.OrderDate
FROM Customers c
OUTER APPLY (
    SELECT TOP 1
        o.OrderID,
        o.OrderDate
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
    ORDER BY o.OrderDate DESC
) x;
```

> Generally used over `LEFT JOIN` when right side needs custom logic per left row, e.g., latest order per customer.

> T-SQL `OUTER APPLY ... <alias>` MYSQL equivalent is `LEFT JOIN LATERAL ... <alias> ON TRUE`

`APPLY` returns table-valued results meanign it returns multiple rows and columns. Scalar functions return ecactly one value. Aggregated functions return one value calculated from multiple rows of groups. 

## Exercise: Combine query results with set operators

https://learn.microsoft.com/en-gb/training/modules/combine-query-results-set-operators/6-knowledge-check

Challenge: use table value function to retrieve custoemr details

```sql
CREATE OR ALTER FUNCTION SalesLT.fn_CustomerAddresses (@CustomerID int)
RETURNS TABLE
RETURN
    SELECT ca.AddressType, a.AddressLine1, a.AddressLine2, a.City, a.StateProvince, a.CountryRegion, a.PostalCode
    FROM SalesLT.CustomerAddress as ca
    JOIN SalesLT.Address AS a
        ON a.AddressID = ca.AddressID
    WHERE ca.CustomerID = @CustomerID;

SELECT c.CustomerID, c.CompanyName, a.*
    FROM SalesLT.Customer AS c
    CROSS APPLY SalesLT.fn_CustomerAddresses(c.CustomerID) AS a
ORDER BY c.CustomerID;
```

