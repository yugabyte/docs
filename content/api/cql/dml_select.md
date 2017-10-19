---
title: SELECT
summary: Retrieves rows from a table.
---

## Synopsis
The `SELECT` statement retrieves (part of) rows of specified columns that meet a given condition from a table. It specifies the columns to be retrieved, the name of the table, and the condition each selected row must satisfy.

## Syntax

### Diagram
<svg class="rrdiagram" version="1.1" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="565" height="145" viewbox="0 0 565 145"><path class="connector" d="M0 22h5m66 0h30m78 0h20m-113 0q5 0 5 5v8q0 5 5 5h88q5 0 5-5v-8q0-5 5-5m5 0h30m28 0h138m-181 0q5 0 5 5v50q0 5 5 5h25m-5 0q-5 0-5-5v-20q0-5 5-5h46m24 0h46q5 0 5 5v20q0 5-5 5m-5 0h25q5 0 5-5v-50q0-5 5-5m5 0h10m54 0h10m91 0h5m-565 95h25m65 0h10m128 0h20m-238 0q5 0 5 5v8q0 5 5 5h213q5 0 5-5v-8q0-5 5-5m5 0h30m54 0h10m117 0h20m-216 0q5 0 5 5v8q0 5 5 5h191q5 0 5-5v-8q0-5 5-5m5 0h5"/><rect class="literal" x="5" y="5" width="66" height="25" rx="7"/><text class="text" x="15" y="22">SELECT</text><rect class="literal" x="101" y="5" width="78" height="25" rx="7"/><text class="text" x="111" y="22">DISTINCT</text><rect class="literal" x="229" y="5" width="28" height="25" rx="7"/><text class="text" x="239" y="22">*</text><rect class="literal" x="290" y="35" width="24" height="25" rx="7"/><text class="text" x="300" y="52">,</text><a xlink:href="../grammar_diagrams#column-name"><rect class="rule" x="249" y="65" width="106" height="25"/><text class="text" x="259" y="82">column_name</text></a><rect class="literal" x="405" y="5" width="54" height="25" rx="7"/><text class="text" x="415" y="22">FROM</text><a xlink:href="../grammar_diagrams#table-name"><rect class="rule" x="469" y="5" width="91" height="25"/><text class="text" x="479" y="22">table_name</text></a><rect class="literal" x="25" y="100" width="65" height="25" rx="7"/><text class="text" x="35" y="117">WHERE</text><a xlink:href="../grammar_diagrams#where-expression"><rect class="rule" x="100" y="100" width="128" height="25"/><text class="text" x="110" y="117">where_expression</text></a><rect class="literal" x="278" y="100" width="54" height="25" rx="7"/><text class="text" x="288" y="117">LIMIT</text><a xlink:href="../grammar_diagrams#limit-expression"><rect class="rule" x="342" y="100" width="117" height="25"/><text class="text" x="352" y="117">limit_expression</text></a></svg>

### Grammar
```
select ::= SELECT [ DISTINCT ] { '*' | column_name [ ',' column_name ... ] } 
               FROM table_name
               [ WHERE where_expression ]
               [ LIMIT limit_expression ]
```
Where

- `table_name` and `column_name` are identifiers (`table_name` may be qualified with a keyspace name).
- `limit_expression` is an integer literal (or a bind variable marker for prepared statements).
- Restrictions for `where_expression` are discussed in the Semantics section below.
- See [Expressions](..#expressions) for more information on syntax rules.

## Semantics
 - An error is raised if the specified `table_name` does not exist.
 - `SELECT DISTINCT` can only be used for partition columns or static columns.
 - `*` means all columns of the table will be retrieved.
 - `LIMIT` clause sets the maximum number of results (rows) to be returned.

### `WHERE` Clause
 - The `where_expression` and `if_expression` must evaluate to boolean values.
 - The `where_expression` can only use `AND` and comparison operators. Other operators are not yet supported.
 - The `where_expression` can specify conditions any column. 
   - Only `=`, `IN` and `NOT IN` operators can be used for conditions on partition columns. 
   - Only operators `=`, `<`, `<=`, `>`, `>=`, `IN` and `NOT IN` can be used for conditions on clustering columns.
   - All logical and boolean operators can be used for conditions on regular columns.

{{< note title="Note" >}}
While the where clause allows a wide range of operators, the exact conditions used in the where clause have significant performance considerations (especially for large datasets).
Some best practices are:

- Use equality conditions on all partition columns (to fix the value of the partition key).
- Use comparison operators on the clustering columns (tighter restrictions are more valuable for left-most clustering columns).
- Generally, the closer a column is to the beginning of the primary key, the higher the performance gain for setting tighter restrictions on it. 

Ideally, these performance considerations should be taken into account when creating the table schema.
{{< /note >}}

## Examples
### Select all rows from a table

``` sql
cqlsh:example> CREATE TABLE employees(department_id INT, 
                                      employee_id INT, 
                                      dept_name TEXT STATIC,
                                      employee_name TEXT, 
                                      PRIMARY KEY(department_id, employee_id));
cqlsh:example> INSERT INTO employees(department_id, employee_id, dept_name, employee_name) 
                   VALUES (1, 1, 'Accounting', 'John');
cqlsh:example> INSERT INTO employees(department_id, employee_id, dept_name, employee_name) 
                   VALUES (1, 2, 'Accounting', 'Jane');
cqlsh:example> INSERT INTO employees(department_id, employee_id, dept_name, employee_name) 
                   VALUES (1, 3, 'Accounting', 'John');
cqlsh:example> INSERT INTO employees(department_id, employee_id, dept_name, employee_name) 
                   VALUES (2, 1, 'Marketing', 'Joe');
cqlsh:example> SELECT * FROM employees;

 department_id | employee_id | dept_name  | employee_name
---------------+-------------+------------+---------------
             2 |           1 |  Marketing |           Joe
             1 |           1 | Accounting |          John
             1 |           2 | Accounting |          Jane
             1 |           3 | Accounting |          John
```

### Select with limit

``` sql
cqlsh:example> SELECT * FROM employees LIMIT 2;

 department_id | employee_id | dept_name  | employee_name
---------------+-------------+------------+---------------
             2 |           1 |  Marketing |           Joe
             1 |           1 | Accounting |          John
```

### Select distinct values

``` sql
cqlsh:example> SELECT DISTINCT dept_name FROM employees;

 dept_name
------------
  Marketing
 Accounting
```

### Select with a condition on the partitioning column

``` sql
cqlsh:example> SELECT * FROM employees WHERE department_id = 2;

 department_id | employee_id | dept_name | employee_name
---------------+-------------+-----------+---------------
             2 |           1 | Marketing |           Joe
```

### Select with condition on the clustering column

``` sql
cqlsh:example> SELECT * FROM employees WHERE department_id = 1 AND employee_id <= 2;

 department_id | employee_id | dept_name  | employee_name
---------------+-------------+------------+---------------
             1 |           1 | Accounting |          John
             1 |           2 | Accounting |          Jane
```

### Select with condition on a regular column

``` sql
cqlsh:example> SELECT * FROM employees WHERE department_id = 1 AND employee_name = 'John';

 department_id | employee_id | dept_name  | employee_name
---------------+-------------+------------+---------------
             1 |           1 | Accounting |          John
             1 |           3 | Accounting |          John
```

## See Also

[`CREATE TABLE`](../ddl_create_table)
[`INSERT`](../dml_insert)
[`SELECT`](../dml_select)
[`UPDATE`](../dml_update)
[`Expression`](..#expressions)
[Other CQL Statements](..)