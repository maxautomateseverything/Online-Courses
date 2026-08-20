# Transform data by implementing pivot, unpivot, rollup, and cube

T-SQL allows transforming and grouping of data to meet requirements.

Done through operators such as:
- PIVOT
- UNPIVOT
- CUBE
- ROLLUP
- GROUPING SETS

Learn:
- Write queries that pivot adn unpivot result sets.
- Write queries that specify multiple groups with GROUPING SETS, CUBE and ROLLUP.

## Write queries that pivot and unpivot result sets:

PIVOT rorates from row based to column based orientation - consolidates values in a column to a list of distinct values and projects the list across as column headers and aggregate the values in the new columns.

E.g., pivoting on Category column contining duplicate values woudl be converted to be distinct column headers:

| Category | Column A | Column B |
|----------|----------|----------|
|Category A | ... | ... | 
|Category B | ... | ... | 
|Category B | ... | ... | 
|Category A | ... | ... | 
|Category A | ... | ... | 
|Category B | ... | ... | 
|Category B | ... | ... | 
|Category A | ... | ... | 

Becomes:

| Category A | Category B |
|----------|----------|
| ... | ... | 

Each distinct category is created as a column header and the values will be aggregated beneath.

### Use PIVOT and pivot a result set:

Pivot a result set using PIVOT that opeartes on the output of the FROM clause in a SELECT statement.

PIVOT requries three elements:
- Aggregation: performed on the grouped rows and determines the value of the cells (e.g., SUM).
- Spreading: outlining which values should become columns using sqaure brackets to signal a column identifier.
- Grouping: PIVOT groups by any column that is not being used for aggregation or for creating new pivoted columns.

> You also need to assign the table an alias.

```sql
SELECT  Category, [2019],[2020],[2021]
FROM  ( SELECT  Category, Qty, Orderyear FROM Sales.CategoryQtyYear) AS D 
          PIVOT(SUM(qty) FOR orderyear IN ([2019],[2020],[2021])) AS pvt;
```
- `orderyear` is the column providing the spreading values
- `qty` is the column used in aggregation

### Use UNPIVOT to unpivot a result set:

The reverse of pivoting where we turn columns into rows - useful for converting pivoted data into row-orientated tabular display.

`UNPIVOT` requires three elements:
- source of columns to be unpivoted
- name of new column that will display unpivoted values
- name of column that will display names of unpivoted values

```sql
SELECT category, qty, orderyear
FROM Sales.PivotedCategorySales
UNPIVOT(qty FOR orderyear IN([2019],[2020],[2021])) AS unpvt;
```

- `2019, 2020 and 2021` are columns to be unpivoted.
- use new column name `orderyear`.
- `qty` values will be displayed in new qty column.

> One of more columns can be defined as sources to be converted into rows - data will be spread into one or more rows depending on how many columns are unpivoted.

In the example, each `orderyear` value will be copied into a new row and associated with its `category` value - any NULLs are removed/not created as a row.

Example shown as we turn this:

| Category	| 2019	| 2020	| 2021 |
|---|---|---|---|
| Beverages	| 1842	| 3996	| 3694 |
| Condiments |	962	| 2895	| 1441 |
| Confections	| 1357	| 4137	| 2412 |
| Dairy Products |	2086	| 4374	| 2689 |
|...|...|...|...|

Into:

|category | qty| orderyear |
|---|---|---|
| Beverages | 1842 | 2019 |
| Beverages | 3996 | 2020 |
| Beverages | 3694 | 2021 |
| Condiments | 962 | 2019 |
| Condiments | 2895 | 2020 |
|...|...|...|

Where a row is createed at each intersection of `category` and `orderyear`.

> UNPIVOTING does not restore original data - detail level data was lost during aggregation in original PIVOT.

## Write queries that specify multiple groupings with grouping sets:

GROUP BY supports normal grouping, typically for aggregation - however, if different attribute groupings were needed at the same time, e.g., report at different levels, you would normally need multiple queries combined with UNION ALL.

GROUPING SETS in the GROUP BY clause provides a solution to produce aggregates of multiple groupings in the same query.

### Use the GROUPING SETS subclause:

Use GROUPING SETS by specifying the combinations of attributes on which to group:

```sql
SELECT <column list with aggregate(s)>
FROM <source>
GROUP BY 
GROUPING SETS(
    (<column_name>),--one or more columns
    (<column_name>),--one or more columns
    () -- empty parentheses if aggregating all rows
        );
```

For example, aggregate Category and Cust columns, in addition to empty parentheses notation to aggregate all rows:

```sql
SELECT Category, Cust, SUM(Qty) AS TotalQty
FROM Sales.CategorySales
GROUP BY 
    GROUPING SETS((Category),(Cust),())
ORDER BY Category, Cust;
```

NULLs may be returned in the result set either because it was stored in the underlying source or because it is a palceholder in a row generated as an aggregate result.

> Use the GROUPING() function to know whether the NULL comes form the underlying data or is a placeholder - it returns 1 for a placeholder and 0 for actual NULL. GROUPING_ID() extends this to compute the goruping level across multiple columns.

### Use the CUBE and ROLLUP subclauses:

CUBE and ROLLUP subclauses also enable multiple groupings for aggregating data - but they do not need you to specify each set of attributes to group.

For a given set of columns:
- CUBE will determine all possible combinations and output groupins.
- ROLLUP creates combinations assuming the input columns represent a hierarchy.

> CUBE and ROLLUP can be thought of as shortcuts to GROUPING SETS.

To use CUBE you append it to the GROUP BY clause then provide a list of columns to group:

```sql
SELECT Category, Cust, SUM(Qty) AS TotalQty
FROM Sales.CategorySales
GROUP BY CUBE(Category,Cust);
```
This would result in the combinations (Category, Cust) and (Cust, Category)

To use ROLLUP you append it to the GROUP BY clause then provide a list of columns to group:

```sql
SELECT Category, Subcategory, Product, SUM(Qty) AS TotalQty
FROM Sales.ProductSales
GROUP BY ROLLUP(Category,Subcategory, Product);
```
This would result in the combinations (Category, Subcategory, Product), (Category, Subcategory), and (Category), and the ahhrehate on all empty ().

The order in which columns are supplied matters in ROLLUP as it assumes the columns are listed in an order that expresses a hierarchy - it provides subtotals for each groupins along with a grand total for all groupings at the end.

## Exercise: Pivoting and grouping sets

Challenge 1: Count product colours by category

```sql
SELECT *
FROM (
    SELECT 
        P.ProductId, 
        PC.Name AS Category, 
        ISNULL(P.Colour, 'Uncoloured') AS Color
    FROM Saleslt.ProductCategory AS PC
    JOIN Saleslt.Product AS P
        ON P.ProductCategoryID = P.ProductCategoryID
    ) AS ProductColours
PIVOT (
    COUNT(ProductId) FOR Colour IN (
        [Red], [Blue], [Black], [Silver], [Yellow], [Grey], [Multi], [Uncoloured])
    ) AS ColourCountsByCategory
ORDER BY Category
```

Challenge 2: Aggregate sales data by product and sales person

```sql
SELECT Product, SalesPerson, SUM(TotalDue) AS TotalSales
FROM Saleslt.v_ProductSales
GROUP BY ROLLUP (Product, SalesPerson)
```