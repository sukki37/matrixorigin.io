# **SELECT**

## **Description**

Retrieves data from a table.

## **Syntax**

``` sql
SELECT
    [ALL | DISTINCT ]
    select_expr [, select_expr] [[AS] alias] ...
    [INTO variable [, ...]]
    [FROM table_references
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC]] [ NULLS { FIRST | LAST } ]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
```

### Explanations

The most commonly used clauses of `SELECT` statements are these:

#### `select_expr`

Each `select_expr` indicates a column that you want to retrieve. There must be at least one `select_expr`.

The list of `select_expr` terms comprises the select list that indicates which columns to retrieve. Terms specify a column or expression or can use *-shorthand:

- A select list consisting only of a single unqualified * can be used as shorthand to select all columns from all tables:

```
SELECT * FROM t1
```

- `tbl_name.*` can be used as a qualified shorthand to select all columns from the named table:

```
SELECT t1.*, t2.* FROM t1
```

- A `select_expr` can be given an alias using `AS` alias_name.

#### `table_references`

- The `FROM table_references` clause indicates the table or tables from which to retrieve rows.

- You can refer to a table within the default database as `tbl_name`, or as `db_name.tbl_name` to specify a database explicitly. You can refer to a column as `col_name`, `tbl_name.col_name`, or `db_name.tbl_name.col_name`. You need not specify a `tbl_name` or `db_name.tbl_name` prefix for a column reference unless the reference would be ambiguous.

- A table reference can be aliased using `tbl_name AS alias_name` or `tbl_name alias_name`.

#### `WHERE`

The `WHERE` clause, if given, indicates the condition or conditions that rows must satisfy to be selected. `where_condition` is an expression that evaluates to true for each row to be selected. The statement selects all rows if there is no `WHERE` clause.

#### `GROUP BY`

Columns selected for output can be referred to in ORDER BY and GROUP BY clauses using column names, column aliases, or column positions.

#### `HAVING`

The `HAVING` clause, like the `WHERE` clause, specifies selection conditions.

#### `ORDER BY`

To sort in reverse order, add the DESC (descending) keyword to the name of the column in the `ORDER BY` clause that you are sorting by. The default is ascending order; this can be specified explicitly using the ASC keyword.

#### `LIMIT`

The `LIMIT` clause can be used to constrain the number of rows returned by the SELECT statement.

## **Examples**

```sql
create table t1 (spID int,userID int,score smallint);
insert into t1 values (1,1,1);
insert into t1 values (2,2,2);
insert into t1 values (2,1,4);
insert into t1 values (3,3,3);
insert into t1 values (1,1,5);
insert into t1 values (4,6,10);
insert into t1 values (5,11,99);
insert into t1 values (null,0,99);

mysql> SELECT * FROM t1 WHERE spID>2 AND userID <2 || userID >=2 OR userID < 2 LIMIT 3;
+------+--------+-------+
| spid | userid | score |
+------+--------+-------+
| NULL |      0 |    99 |
|    1 |      1 |     1 |
|    2 |      2 |     2 |
+------+--------+-------+

mysql> SELECT userID,MAX(score) max_score FROM t1 WHERE userID <2 || userID > 3 GROUP BY userID ORDER BY max_score;
+--------+-----------+
| userid | max_score |
+--------+-----------+
|      1 |         5 |
|      6 |        10 |
|      0 |        99 |
|     11 |        99 |
+--------+-----------+

mysql> select userID,count(score) from t1 group by userID having count(score)>1 order by userID;
+--------+--------------+
| userid | count(score) |
+--------+--------------+
|      1 |            3 |
+--------+--------------+

mysql> select userID,count(score) from t1 where userID>2 group by userID having count(score)>1 order by userID;
Empty set (0.01 sec)s

mysql> select * from t1 order by spID asc nulls last;
+------+--------+-------+
| spid | userid | score |
+------+--------+-------+
|    1 |      1 |     1 |
|    1 |      1 |     5 |
|    2 |      2 |     2 |
|    2 |      1 |     4 |
|    3 |      3 |     3 |
|    4 |      6 |    10 |
|    5 |     11 |    99 |
| NULL |      0 |    99 |
+------+--------+-------+
```

## **Constraints**

1. Table alias is not supported in GROUP BY.
2. SELECT...FOR UPDATE clause is not supported now.
3. INTO OUTFILE is limitedly support.
