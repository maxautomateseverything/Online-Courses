# Write queries that use window functions

Allow you to define a subset of rows and apply functions against those rows.

Can be used in place of constructs such as self joins or temporary tables.

Learn:
- Describe window functions
- Use `OVER` clause
- Use `RANK`, `AGGREGATE`, `OFFSET` functions

## Describe window functions:

Require a set of rows, known as a window, to apply the window function on - `OVER` is used to define that window.

Enable solutions for row numbers or running totals, as well as efficeint ways to compare values without a join.

Provide unique functionality that is hard to replciate:
- order rows without affecting sort order of output
- apply separate window functions to a divided result set
- subdividing a partition by defining upper and lower boundaries of window frame

---

## Use `OVER` clause:

Defines the window (rows) that the function is applied to - all, subset, specific order?

`OVER` clause takes the following arguments:
- Which group of rows? - `PARTITION BY`: splits rows into groups.
- In which order? - `ORDER BY`: establishes an order inside each partition.
- Which rows around the current row? - `ROWS / RANGE`: defines the window frame.

> Using `ORDER BY` without explicitly defining a frame, behaviour defaults to the equivalent of writing `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` meaning the range from the start of the first row in the partition to the current row.

> If `ORDER BY` is not passed as an argument, the entire partition is used as a frame.

> If `OVER` clause is not used the window function will be applied to the entuire result set.

![alt text](image.png)



