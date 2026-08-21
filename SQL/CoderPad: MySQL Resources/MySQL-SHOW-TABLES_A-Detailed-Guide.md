# MySQL SHOW TABLES: A Detailed Guide

https://coderpad.io/blog/development/mysql-show-tables-a-detailed-guide/



## Command definition:

Used to list tables in MySQL database.


## Why it's useful:

A GUI often displas tables inside a database through trees, but you may not always have access to a GUI. If only command line is available `show tables` is useful.

It is flexible as it allows you to combine it with WHERE and LIKE clauses to filter the output.

It can be used in scripts so you are able to automate some tasks that would not be possible using a GUI.


## How to use it:

### `SHOW TABLE`:

```sql
SHOW tables
```

Will simply return a single column list of the tables in the database.

### `SHOW FULL TABLE`:

`SHOW tables`, despite the name, does list views as well as tables. When using `SHOW tables` only, there is no way to distinguish between views and tables.

`SHOW full tables` outputs not only tabe names but also table types allowing you to distinguish between base tables and views.

### `LIKE` clause:

Subset of tables can be outputted using the LIKE clause. For example:

```sql
SHOW tables LIKE 'stu%';
```

### `WHERE` clause:

MySQL returns the list of tables/views in tabular form. The column that contains the list of tables is called 'Tables_in_<database_name>'.

We can use that name to filter the column using a WHERE clause, for example:

```sql
SHOW tables WHERE length (<database_name>) <= 8;

SHOW tables WHERE <database_name> = 'students';
```

> In these commands you can use `SHOW FULL TABLES` as well.

### Tables from different DB:

You can retrieve the list of tables from another database while in one database.

```sql
SHOW tables FROM <other_database_name>
```

> You must have necessary privalages to run SHOW tables.


## Alternatives to SHOW tables:

SHOW tables is non-standard and unique to MySQL - i.e., other DB engines dont use such syntax.

A more transferable query, to at least some other RDBMSs, is INFORMATION_SCHEMA views - a ANSI-standard set of read-only views that provide information about all ttables, views, columns and procedures in a DB.

```sql
SELECT *
FROM information_schema.tables
WHERE ...
```

> It works in the same way as a normal select query where you can specify the columns and apply filtering clauses.

The information_schema view contains alot more information than just table names - there are views for columns, indexes, views and other database artifacts.

```sql
SELECT *
FROM information_schema.columns
```
