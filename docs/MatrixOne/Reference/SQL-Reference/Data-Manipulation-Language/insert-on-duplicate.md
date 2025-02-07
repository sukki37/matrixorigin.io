# **INSERT ... ON DUPLICATE KEY UPDATE**

## **Description**

`INSERT ... ON DUPLICATE KEY UPDATE` is used to insert data into the database table; if the data already exists, update the data; otherwise, insert new data.

The `INSERT INTO` statement is a standard statement used to insert data into a database table; the `ON DUPLICATE KEY UPDATE` statement performs an update operation when there are duplicate records in the table. If a record with the same unique index or primary key exists in the table, use the `UPDATE` clause to update the corresponding column value; otherwise, use the `INSERT` clause to insert a new record.

It should be noted that the premise of using this syntax is that a primary key constraint needs to be established in the table to determine whether there are duplicate records. At the same time, both the update operation and the insert operation need to set the corresponding column value. Otherwise, a syntax error will result.

## **Syntax**

```
> INSERT INTO [db.]table [(c1, c2, c3)] VALUES (v11, v12, v13), (v21, v22, v23), ...
  [ON DUPLICATE KEY UPDATE column1 = value1, column2 = value2, column3 = value3, ...];
```

## **Examples**

```sql
CREATE TABLE user (
    id INT(11) NOT NULL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    age INT(3) NOT NULL
);
-- Insert a new data; it does not exist, insert the new data
INSERT INTO user (id, name, age) VALUES (1, 'Tom', 18)
ON DUPLICATE KEY UPDATE name='Tom', age=18;

mysql> select * from user;
+------+------+------+
| id   | name | age  |
+------+------+------+
|    1 | Tom  |   18 |
+------+------+------+
1 row in set (0.01 sec)

-- Increment the age field of an existing record by 1 while keeping the name field unchanged
INSERT INTO user (id, name, age) VALUES (1, 'Tom', 18)
ON DUPLICATE KEY UPDATE age=age+1;

mysql> select * from user;
+------+------+------+
| id   | name | age  |
+------+------+------+
|    1 | Tom  |   19 |
+------+------+------+
1 row in set (0.00 sec)

-- Insert a new record, and update the name and age fields to the specified values
INSERT INTO user (id, name, age) VALUES (2, 'Lucy', 20)
ON DUPLICATE KEY UPDATE name='Lucy', age=20;

mysql> select * from user;
+------+------+------+
| id   | name | age  |
+------+------+------+
|    1 | Tom  |   19 |
|    2 | Lucy |   20 |
+------+------+------+
2 rows in set (0.01 sec)
```

## **Constraints**

1. `INSERT ... ON DUPLICATE KEY UPDATE` only supports 1 unique constraint and does not support multiple unique constraints.
2. Unique key are not currently supported with `INSERT ... ON DUPLICATE KEY UPDATE`, and since unique key can be null, some unknown errors can occur.
