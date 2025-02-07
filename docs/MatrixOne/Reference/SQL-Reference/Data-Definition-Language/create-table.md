# **CREATE TABLE**

## **Description**

Create a new table.

## **Syntax**

```
> CREATE [TEMPORARY] TABLE [IF NOT EXISTS] [db.]table_name [comment = "comment of table"];
(
    name1 type1 [comment 'comment of column'] [AUTO_INCREMENT] [[PRIMARY] KEY] [[FOREIGN] KEY],
    name2 type2 [comment 'comment of column'],
    ...
)
    [cluster by (column_name1, column_name2, ...);]
    [partition_options]
```

### Explanations

#### Temporary Tables

You can use the `TEMPORARY` keyword when creating a table. A `TEMPORARY` table is visible only within the current session, and is dropped automatically when the session is closed. This means that two different sessions can use the same temporary table name without conflicting with each other or with an existing non-TEMPORARY table of the same name. (The existing table is hidden until the temporary table is dropped.)

Dropping a database does automatically drop any `TEMPORARY` tables created within that database.

The creating session can perform any operation on the table, such as `DROP TABLE`, `INSERT`, `UPDATE`, or `SELECT`.

#### COMMENT

A comment for a column or a table can be specified with the `COMMENT` option.

- Up to 1024 characters long. The comment is displayed by the `SHOW CREATE TABLE` and `SHOW FULL COLUMNS` statements. It is also shown in the `COLUMN_COMMENT` column of the `INFORMATION_SCHEMA.COLUMNS` table.

#### AUTO_INCREMENT

The initial `AUTO_INCREMENT` value for the table.

An integer column can have the additional attribute `AUTO_INCREMENT`. When you insert a value of NULL (recommended) or 0 into an indexed AUTO_INCREMENT column, the column is set to the next sequence value. Typically this is value+1, where value is the largest value for the column currently in the table. AUTO_INCREMENT sequences begin with 1.

There can be only one `AUTO_INCREMENT` column per table, it must be indexed, and it cannot have a DEFAULT value. An `AUTO_INCREMENT` column works properly only if it contains only positive values. Inserting a negative number is regarded as inserting a very large positive number. This is done to avoid precision problems when numbers "wrap" over from positive to negative and also to ensure that you do not accidentally get an `AUTO_INCREMENT` column that contains 0.

#### PRIMARY KEY

The PRIMARY KEY constraint uniquely identifies each record in a table.

Primary keys must contain UNIQUE values, and cannot contain NULL values.

A table can have only ONE primary key; and in the table, this primary key can consist of single column (field).

- **SQL PRIMARY KEY on CREATE TABLE**

The following SQL creates a PRIMARY KEY on the "ID" column when the "Persons" table is created:

```
> CREATE TABLE Persons (
    ID int NOT NULL,
    LastName varchar(255) NOT NULL,
    FirstName varchar(255),
    Age int,
    PRIMARY KEY (ID)
);
```

#### FOREIGN KEY

The FOREIGN KEY constraint is used to prevent actions that would destroy links between tables.

A FOREIGN KEY is a field (or collection of fields) in one table, that refers to the PRIMARY KEY in another table.

The table with the foreign key is called the child table, and the table with the primary key is called the referenced or parent table.

The FOREIGN KEY constraint prevents invalid data from being inserted into the foreign key column, because it has to be one of the values contained in the parent table.

When defining FOREIGN KEY, the following rules need to be followed:

- The parent table must already exist in the database or be a table currently being created. In the latter case, the parent table and the slave table are the same table, such a table is called a self-referential table, and this structure is called self-referential integrity.

- A primary key must be defined for the parent table.

- Specify the column name or combination of column names after the table name of the parent table. This column or combination of columns must be the primary or candidate key of the primary table. Currently, MatrixOne only supports single-column foreign key constraints.

- The number of columns in the foreign key must be the same as the number of columns in the primary key of the parent table.

- The data type of the column in the foreign key must be the same as the data type of the corresponding column in the primary key of the parent table.

The following is an example to illustrate the association of parent and child tables through FOREIGN KEY and PRIMARY KEY:

First, create a parent table with field a as the primary key:

```sql
create table t1(a int primary key,b varchar(5));
insert into t1 values(101,'abc'),(102,'def');
mysql> select * from t1;
+------+------+
| a    | b    |
+------+------+
|  101 | abc  |
|  102 | def  |
+------+------+
2 rows in set (0.00 sec)
```

Then create a child table with field c as the foreign key, associated with parent table field a:

```sql
create table t2(a int ,b varchar(5),c int, foreign key(c) references t1(a));
insert into t2 values(1,'zs1',101),(2,'zs2',102);
insert into t2 values(3,'xyz',null);
mysql> select * from t2;
+------+------+------+
| a    | b    | c    |
+------+------+------+
|    1 | zs1  |  101 |
|    2 | zs2  |  102 |
|    3 | xyz  | NULL |
+------+------+------+
3 rows in set (0.00 sec)
```

For more information on data integrity constraints, see [Data Integrity Constraints Overview](../../../Develop/schema-design/data-integrity/overview-of-integrity-constraint-types.md).

#### Cluster by

`Cluster by` is a command used to optimize the physical arrangement of a table. When creating a table, the `Cluster by` command can physically sort the table based on a specified column for tables without a primary key. It will rearrange the data rows to match the order of values in that column. Using `Cluster by` improves query performance.

- The syntax for a single column is: `create table() cluster by col;`
- The syntax for multiple columns is: `create table() cluster by (col1, col2);`

__Note:__ `Cluster by` cannot coexist with a primary key, or a syntax error will occur. `Cluster by` can only be specified when creating a table and does not support dynamic creation.

For more information on using `Cluster by` for performing tuning, see [Using `Cluster by` for performance tuning](../../../Performance-Tuning/optimization-concepts/through-cluster-by.md).

#### Table PARTITION and PARTITIONS

```
partition_options:
 PARTITION BY
     { [LINEAR] HASH(expr)
     | [LINEAR] KEY [ALGORITHM={1 | 2}] (column_list)
     | RANGE{(expr) | COLUMNS(column_list)}
     | LIST{(expr) | COLUMNS(column_list)} }
 [PARTITIONS num]
 [SUBPARTITION BY
     { [LINEAR] HASH(expr)
     | [LINEAR] KEY [ALGORITHM={1 | 2}] (column_list) }
 ]
 [(partition_definition [, partition_definition] ...)]

partition_definition:
 PARTITION partition_name
     [VALUES
         {LESS THAN {(expr | value_list) | MAXVALUE}
         |
         IN (value_list)}]
```

Partitions can be modified, merged, added to tables, and dropped from tables.

- **PARTITION BY**

If used, a partition_options clause begins with PARTITION BY. This clause contains the function that is used to determine the partition; the function returns an integer value ranging from 1 to num, where num is the number of partitions.

- **HASH(expr)**

Hashes one or more columns to create a key for placing and locating rows. expr is an expression using one or more table columns. For example, these are both valid CREATE TABLE statements using PARTITION BY HASH:

```
CREATE TABLE t1 (col1 INT, col2 CHAR(5))
    PARTITION BY HASH(col1);

CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATETIME)
    PARTITION BY HASH ( YEAR(col3) );
```

- **KEY(column_list)**

This is similar to `HASH`. The column_list argument is simply a list of 1 or more table columns (maximum: 16). This example shows a simple table partitioned by key, with 4 partitions:

```
CREATE TABLE tk (col1 INT, col2 CHAR(5), col3 DATE)
    PARTITION BY KEY(col3)
    PARTITIONS 4;
```

For tables that are partitioned by key, you can employ linear partitioning by using the `LINEAR` keyword. This has the same effect as with tables that are partitioned by `HASH`. This example uses linear partitioning by `key` to distribute data between 5 partitions:

```
CREATE TABLE tk (col1 INT, col2 CHAR(5), col3 DATE)
    PARTITION BY LINEAR KEY(col3)
    PARTITIONS 5;
```

- **RANGE(expr)**

In this case, expr shows a range of values using a set of `VALUES LESS THAN` operators. When using range partitioning, you must define at least one partition using `VALUES LESS THAN`. You cannot use `VALUES IN` with range partitioning.

`PARTITION ... VALUES LESS THAN ...` statements work in a consecutive fashion. `VALUES LESS THAN MAXVALUE` works to specify "leftover" values that are greater than the maximum value otherwise specified.

The clauses must be arranged in such a way that the upper limit specified in each successive `VALUES LESS THAN` is greater than that of the previous one, with the one referencing `MAXVALUE` coming last of all in the list.

- **RANGE COLUMNS(column_list)**

This variant on `RANGE` facilitates partition pruning for queries using range conditions on multiple columns (that is, having conditions such as `WHERE a = 1 AND b < 10 or WHERE a = 1 AND b = 10 AND c < 10)`. It enables you to specify value ranges in multiple columns by using a list of columns in the COLUMNS clause and a set of column values in each `PARTITION ... VALUES LESS THAN (value_list)` partition definition clause. (In the simplest case, this set consists of a single column.) The maximum number of columns that can be referenced in the column_list and value_list is 16.

The column_list used in the `COLUMNS` clause may contain only names of columns; each column in the list must be one of the following MySQL data types: the integer types; the string types; and time or date column types. Columns using BLOB, TEXT, SET, ENUM, BIT, or spatial data types are not permitted; columns that use floating-point number types are also not permitted. You also may not use functions or arithmetic expressions in the `COLUMNS` clause.

The `VALUES LESS THAN` clause used in a partition definition must specify a literal value for each column that appears in the COLUMNS() clause; that is, the list of values used for each `VALUES LESS THAN` clause must contain the same number of values as there are columns listed in the `COLUMNS` clause. An attempt to use more or fewer values in a `VALUES LESS THAN` clause than there are in the COLUMNS clause causes the statement to fail with the error Inconsistency in usage of column lists for partitioning.... You cannot use `NULL` for any value appearing in `VALUES LESS THAN`. It is possible to use MAXVALUE more than once for a given column other than the first, as shown in this example:

```
CREATE TABLE rc (
    a INT NOT NULL,
    b INT NOT NULL
)
PARTITION BY RANGE COLUMNS(a,b) (
    PARTITION p0 VALUES LESS THAN (10,5),
    PARTITION p1 VALUES LESS THAN (20,10),
    PARTITION p2 VALUES LESS THAN (50,MAXVALUE),
    PARTITION p3 VALUES LESS THAN (65,MAXVALUE),
    PARTITION p4 VALUES LESS THAN (MAXVALUE,MAXVALUE)
);
```

Each value used in a `VALUES LESS THAN` value list must match the type of the corresponding column exactly; no conversion is made. For example, you cannot use the string '1' for a value that matches a column that uses an integer type (you must use the numeral 1 instead), nor can you use the numeral 1 for a value that matches a column that uses a string type (in such a case, you must use a quoted string: '1').

- **LIST(expr)**

This is useful when assigning partitions based on a table column with a restricted set of possible values, such as a state or country code. In such a case, all rows pertaining to a certain state or country can be assigned to a single partition, or a partition can be reserved for a certain set of states or countries. It is similar to RANGE, except that only VALUES IN may be used to specify permissible values for each partition.

`VALUES IN` is used with a list of values to be matched. For instance, you could create a partitioning scheme such as the following:

```
CREATE TABLE client_firms (
    id   INT,
    name VARCHAR(35)
)
PARTITION BY LIST (id) (
    PARTITION r0 VALUES IN (1, 5, 9, 13, 17, 21),
    PARTITION r1 VALUES IN (2, 6, 10, 14, 18, 22),
    PARTITION r2 VALUES IN (3, 7, 11, 15, 19, 23),
    PARTITION r3 VALUES IN (4, 8, 12, 16, 20, 24)
);
```

When using list partitioning, you must define at least one partition using VALUES IN. You cannot use VALUES LESS THAN with PARTITION BY LIST.

!!! note
     For tables partitioned by LIST, the value list used with VALUES IN must consist of integer values only. In MySQL 8.0, you can overcome this limitation using partitioning by LIST COLUMNS, which is described later in this section.

- **LIST COLUMNS(column_list)**

This variant on `LIST` facilitates partition pruning for queries using comparison conditions on multiple columns (that is, having conditions such as `WHERE a = 5 AND b = 5 or WHERE a = 1 AND b = 10 AND c = 5`). It enables you to specify values in multiple columns by using a list of columns in the COLUMNS clause and a set of column values in each `PARTITION ... VALUES IN` (value_list) partition definition clause.

The rules governing regarding data types for the column list used in `LIST COLUMNS(column_list)` and the value list used in `VALUES IN(value_list)` are the same as those for the column list used in `RANGE COLUMNS(column_list)` and the value list used in `VALUES LESS THAN(value_list)`, respectively, except that in the `VALUES IN` clause, `MAXVALUE` is not permitted, and you may use `NULL`.

There is one important difference between the list of values used for VALUES IN with `PARTITION BY LIST COLUMNS` as opposed to when it is used with `PARTITION BY LIST`. When used with `PARTITION BY LIST COLUMNS`, each element in the `VALUES IN` clause must be a set of column values; the number of values in each set must be the same as the number of columns used in the `COLUMNS` clause, and the data types of these values must match those of the columns (and occur in the same order). In the simplest case, the set consists of a single column. The maximum number of columns that can be used in the column_list and in the elements making up the value_list is 16.

The table defined by the following `CREATE TABLE` statement provides an example of a table using `LIST COLUMNS` partitioning:

```
CREATE TABLE lc (
    a INT NULL,
    b INT NULL
)
PARTITION BY LIST COLUMNS(a,b) (
    PARTITION p0 VALUES IN( (0,0), (NULL,NULL) ),
    PARTITION p1 VALUES IN( (0,1), (0,2), (0,3), (1,1), (1,2) ),
    PARTITION p2 VALUES IN( (1,0), (2,0), (2,1), (3,0), (3,1) ),
    PARTITION p3 VALUES IN( (1,3), (2,2), (2,3), (3,2), (3,3) )
);
```

- **PARTITIONS num**

The number of partitions may optionally be specified with a PARTITIONS num clause, where num is the number of partitions. If both this clause and any PARTITION clauses are used, num must be equal to the total number of any partitions that are declared using PARTITION clauses.

## **Examples**

- Example 1: Create a common table

```sql
CREATE TABLE test(a int, b varchar(10));
INSERT INTO test values(123, 'abc');

mysql> SELECT * FROM test;
+------+---------+
|   a  |    b    |
+------+---------+
|  123 |   abc   |
+------+---------+
```

- Example 2: Add comments when creating a table

```sql
create table t2 (a int, b int) comment = "fact table";

mysql> show create table t2;
+-------+---------------------------------------------------------------------------------------+
| Table | Create Table                                                                          |
+-------+---------------------------------------------------------------------------------------+
| t2    | CREATE TABLE `t2` (
`a` INT DEFAULT NULL,
`b` INT DEFAULT NULL
) COMMENT='fact table',    |
+-------+---------------------------------------------------------------------------------------+
```

- Example 3: Add comments to columns when creating tables

```sql
create table t3 (a int comment 'Column comment', b int) comment = "table";

mysql> SHOW CREATE TABLE t3;
+-------+----------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                             |
+-------+----------------------------------------------------------------------------------------------------------+
| t3    | CREATE TABLE `t3` (
`a` INT DEFAULT NULL COMMENT 'Column comment',
`b` INT DEFAULT NULL
) COMMENT='table',     |
+-------+----------------------------------------------------------------------------------------------------------+
```

- Example 4: Create a common partitioned table

```sql
CREATE TABLE tp1 (col1 INT, col2 CHAR(5), col3 DATE) PARTITION BY KEY(col3) PARTITIONS 4;

mysql> SHOW CREATE TABLE tp1;
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                             |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp1   | CREATE TABLE `tp1` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATE DEFAULT NULL
) partition by key algorithm = 2 (col3) partitions 4 |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- do not specify the number of partitions
CREATE TABLE tp2 (col1 INT, col2 CHAR(5), col3 DATE) PARTITION BY KEY(col3);

mysql> SHOW CREATE TABLE tp2;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------+
| tp2   | CREATE TABLE `tp2` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATE DEFAULT NULL
) partition by key algorithm = 2 (col3) |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- Specify partition algorithm
CREATE TABLE tp3
(
    col1 INT,
    col2 CHAR(5),
    col3 DATE
) PARTITION BY KEY ALGORITHM = 1 (col3);


mysql> show create table tp3;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------+
| tp3   | CREATE TABLE `tp3` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATE DEFAULT NULL
) partition by key algorithm = 1 (col3) |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- Specify partition algorithm and the number of partitions
CREATE TABLE tp4 (col1 INT, col2 CHAR(5), col3 DATE) PARTITION BY LINEAR KEY ALGORITHM = 1 (col3) PARTITIONS 5;

mysql> SHOW CREATE TABLE tp4;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                    |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp4   | CREATE TABLE `tp4` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATE DEFAULT NULL
) partition by linear key algorithm = 1 (col3) partitions 5 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

-- Multi-column partition
CREATE TABLE tp5
(
    col1 INT,
    col2 CHAR(5),
    col3 DATE
) PARTITION BY KEY(col1, col2) PARTITIONS 4;

mysql> SHOW CREATE TABLE tp5;
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                   |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp5   | CREATE TABLE `tp5` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATE DEFAULT NULL
) partition by key algorithm = 2 (col1, col2) partitions 4 |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

-- Create a primary key column partition
CREATE TABLE tp6
(
    col1 INT  NOT NULL PRIMARY KEY,
    col2 DATE NOT NULL,
    col3 INT  NOT NULL,
    col4 INT  NOT NULL
) PARTITION BY KEY(col1) PARTITIONS 4;

mysql> SHOW CREATE TABLE tp6;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                        |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp6   | CREATE TABLE `tp6` (
`col1` INT NOT NULL,
`col2` DATE NOT NULL,
`col3` INT NOT NULL,
`col4` INT NOT NULL,
PRIMARY KEY (`col1`)
) partition by key algorithm = 2 (col1) partitions 4 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

-- Create HASH partition
CREATE TABLE tp7
(
    col1 INT,
    col2 CHAR(5)
) PARTITION BY HASH(col1);

mysql> SHOW CREATE TABLE tp7;
+-------+------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                         |
+-------+------------------------------------------------------------------------------------------------------+
| tp7   | CREATE TABLE `tp7` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL
) partition by hash (col1) |
+-------+------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

-- Specifies the number of HASH partitions when creating hash partition
CREATE TABLE tp8
(
    col1 INT,
    col2 CHAR(5)
) PARTITION BY HASH(col1) PARTITIONS 4;

mysql> SHOW CREATE TABLE tp8;
+-------+-------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                      |
+-------+-------------------------------------------------------------------------------------------------------------------+
| tp8   | CREATE TABLE `tp8` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL
) partition by hash (col1) partitions 4 |
+-------+-------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- specify the partition granularity when creating a partition
CREATE TABLE tp9
(
    col1 INT,
    col2 CHAR(5),
    col3 DATETIME
) PARTITION BY HASH (YEAR(col3));

mysql> SHOW CREATE TABLE tp9;
+-------+------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                             |
+-------+------------------------------------------------------------------------------------------------------------------------------------------+
| tp9   | CREATE TABLE `tp9` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATETIME DEFAULT NULL
) partition by hash (year(col3)) |
+-------+------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- specify the partition granularity and number of partitions when creating a partition
CREATE TABLE tp10
(
    col1 INT,
    col2 CHAR(5),
    col3 DATE
) PARTITION BY LINEAR HASH( YEAR(col3)) PARTITIONS 6;

mysql> SHOW CREATE TABLE tp10;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                              |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp10  | CREATE TABLE `tp10` (
`col1` INT DEFAULT NULL,
`col2` CHAR(5) DEFAULT NULL,
`col3` DATE DEFAULT NULL
) partition by linear hash (year(col3)) partitions 6 |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- Use the primary key column as the HASH partition when creating a partition
CREATE TABLE tp12 (col1 INT NOT NULL PRIMARY KEY, col2 DATE NOT NULL, col3 INT NOT NULL, col4 INT NOT NULL) PARTITION BY HASH(col1) PARTITIONS 4;

mysql> SHOW CREATE TABLE tp12;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                            |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp12  | CREATE TABLE `tp12` (
`col1` INT NOT NULL,
`col2` DATE NOT NULL,
`col3` INT NOT NULL,
`col4` INT NOT NULL,
PRIMARY KEY (`col1`)
) partition by hash (col1) partitions 4 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

-- Create a RANGE partition and divide the partition range
CREATE TABLE tp13 (id INT NOT NULL PRIMARY KEY, fname VARCHAR(30), lname VARCHAR(30), hired DATE NOT NULL DEFAULT '1970-01-01', separated DATE NOT NULL DEFAULT '9999-12-31', job_code INT NOT NULL, store_id INT NOT NULL) PARTITION BY RANGE (id) (PARTITION p0 VALUES LESS THAN (6), PARTITION p1 VALUES LESS THAN (11), PARTITION p2 VALUES LESS THAN (16), PARTITION p3 VALUES LESS THAN (21));

mysql> SHOW CREATE TABLE tp13;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                                                         |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp13  | CREATE TABLE `tp13` (
`id` INT NOT NULL,
`fname` VARCHAR(30) DEFAULT NULL,
`lname` VARCHAR(30) DEFAULT NULL,
`hired` DATE DEFAULT '1970-01-01',
`separated` DATE DEFAULT '9999-12-31',
`job_code` INT NOT NULL,
`store_id` INT NOT NULL,
PRIMARY KEY (`id`)
) partition by range(id) (partition p0 values less than (6), partition p1 values less than (11), partition p2 values less than (16), partition p3 values less than (21)) |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

CREATE TABLE tp14 (id INT NOT NULL, fname VARCHAR(30), lname VARCHAR(30), hired DATE NOT NULL DEFAULT '1970-01-01', separated DATE NOT NULL DEFAULT '9999-12-31', job_code INT, store_id INT) PARTITION BY RANGE ( YEAR(separated) ) ( PARTITION p0 VALUES LESS THAN (1991), PARTITION p1 VALUES LESS THAN (1996), PARTITION p2 VALUES LESS THAN (2001), PARTITION p3 VALUES LESS THAN MAXVALUE);

mysql> SHOW CREATE TABLE tp14;
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                                                                                                                       |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp14  | CREATE TABLE `tp14` (
`id` INT NOT NULL,
`fname` VARCHAR(30) DEFAULT NULL,
`lname` VARCHAR(30) DEFAULT NULL,
`hired` DATE DEFAULT '1970-01-01',
`separated` DATE DEFAULT '9999-12-31',
`job_code` INT DEFAULT NULL,
`store_id` INT DEFAULT NULL
) partition by range(year(separated)) (partition p0 values less than (1991), partition p1 values less than (1996), partition p2 values less than (2001), partition p3 values less than (MAXVALUE)) |
+-------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- Use multiple columns as RANGE partitions and specify the range of partitions
CREATE TABLE tp15 (a INT NOT NULL, b INT NOT NULL) PARTITION BY RANGE COLUMNS(a,b) PARTITIONS 4 (PARTITION p0 VALUES LESS THAN (10,5), PARTITION p1 VALUES LESS THAN (20,10), PARTITION p2 VALUES LESS THAN (50,20), PARTITION p3 VALUES LESS THAN (65,30));

mysql> SHOW CREATE TABLE tp15;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                              |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp15  | CREATE TABLE `tp15` (
`a` INT NOT NULL,
`b` INT NOT NULL
) partition by range columns (a, b) partitions 4 (partition p0 values less than (10, 5), partition p1 values less than (20, 10), partition p2 values less than (50, 20), partition p3 values less than (65, 30)) |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

-- Create LIST partition
CREATE TABLE tp16 (id   INT PRIMARY KEY, name VARCHAR(35), age INT unsigned) PARTITION BY LIST (id) (PARTITION r0 VALUES IN (1, 5, 9, 13, 17, 21), PARTITION r1 VALUES IN (2, 6, 10, 14, 18, 22), PARTITION r2 VALUES IN (3, 7, 11, 15, 19, 23), PARTITION r3 VALUES IN (4, 8, 12, 16, 20, 24));

mysql> SHOW CREATE TABLE tp16;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                          |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp16  | CREATE TABLE `tp16` (
`id` INT DEFAULT NULL,
`name` VARCHAR(35) DEFAULT NULL,
`age` INT UNSIGNED DEFAULT NULL,
PRIMARY KEY (`id`)
) partition by list(id) (partition r0 values in (1, 5, 9, 13, 17, 21), partition r1 values in (2, 6, 10, 14, 18, 22), partition r2 values in (3, 7, 11, 15, 19, 23), partition r3 values in (4, 8, 12, 16, 20, 24)) |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

CREATE TABLE tp17 (id   INT, name VARCHAR(35), age INT unsigned) PARTITION BY LIST (id) (PARTITION r0 VALUES IN (1, 5, 9, 13, 17, 21), PARTITION r1 VALUES IN (2, 6, 10, 14, 18, 22), PARTITION r2 VALUES IN (3, 7, 11, 15, 19, 23), PARTITION r3 VALUES IN (4, 8, 12, 16, 20, 24));

mysql> SHOW CREATE TABLE tp17;
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                      |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp17  | CREATE TABLE `tp17` (
`id` INT DEFAULT NULL,
`name` VARCHAR(35) DEFAULT NULL,
`age` INT UNSIGNED DEFAULT NULL
) partition by list(id) (partition r0 values in (1, 5, 9, 13, 17, 21), partition r1 values in (2, 6, 10, 14, 18, 22), partition r2 values in (3, 7, 11, 15, 19, 23), partition r3 values in (4, 8, 12, 16, 20, 24)) |
+-------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.01 sec)

-- Use multiple columns as LIST partitions
CREATE TABLE tp18 (a INT NULL,b INT NULL) PARTITION BY LIST COLUMNS(a,b) (PARTITION p0 VALUES IN( (0,0), (NULL,NULL) ), PARTITION p1 VALUES IN( (0,1), (0,2), (0,3), (1,1), (1,2) ), PARTITION p2 VALUES IN( (1,0), (2,0), (2,1), (3,0), (3,1) ), PARTITION p3 VALUES IN( (1,3), (2,2), (2,3), (3,2), (3,3) ));

mysql> SHOW CREATE TABLE tp18;
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                                                                                                                                                                                           |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| tp18  | CREATE TABLE `tp18` (
`a` INT DEFAULT NULL,
`b` INT DEFAULT NULL
) partition by list columns (a, b) (partition p0 values in ((0, 0), (null, null)), partition p1 values in ((0, 1), (0, 2), (0, 3), (1, 1), (1, 2)), partition p2 values in ((1, 0), (2, 0), (2, 1), (3, 0), (3, 1)), partition p3 values in ((1, 3), (2, 2), (2, 3), (3, 2), (3, 3))) |
+-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

- Example 5: Primary key auto increment

```sql
drop table if exists t1;
create table t1(a bigint primary key auto_increment,
    b varchar(10));
insert into t1(b) values ('bbb');
insert into t1 values (3, 'ccc');
insert into t1(b) values ('bbb1111');

mysql> select * from t1 order by a;
+------+---------+
| a    | b       |
+------+---------+
|    1 | bbb     |
|    3 | ccc     |
|    4 | bbb1111 |
+------+---------+
3 rows in set (0.01 sec)

insert into t1 values (2, 'aaaa1111');

mysql> select * from t1 order by a;
+------+----------+
| a    | b        |
+------+----------+
|    1 | bbb      |
|    2 | aaaa1111 |
|    3 | ccc      |
|    4 | bbb1111  |
+------+----------+
4 rows in set (0.00 sec)

insert into t1(b) values ('aaaa1111');

mysql> select * from t1 order by a;
+------+----------+
| a    | b        |
+------+----------+
|    1 | bbb      |
|    2 | aaaa1111 |
|    3 | ccc      |
|    4 | bbb1111  |
|    5 | aaaa1111 |
+------+----------+
5 rows in set (0.01 sec)

insert into t1 values (100, 'xxxx');
insert into t1(b) values ('xxxx');

mysql> select * from t1 order by a;
+------+----------+
| a    | b        |
+------+----------+
|    1 | bbb      |
|    2 | aaaa1111 |
|    3 | ccc      |
|    4 | bbb1111  |
|    5 | aaaa1111 |
|  100 | xxxx     |
|  101 | xxxx     |
+------+----------+
7 rows in set (0.00 sec)
```

## **Constraints**

`DROP PRIMARY KEY` with `ALTER TABLE` is not supported yet.
