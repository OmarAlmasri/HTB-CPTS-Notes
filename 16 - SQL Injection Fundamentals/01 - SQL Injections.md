## SQL Injection: Operator Precedence & Authentication Bypass
### Core Concept

In SQL, logical operators follow a strict hierarchy of precedence. The **`AND`** operator takes priority over the **`OR`** operator and is evaluated first.
### Mechanism

Consider the standard SQL injection payload:

```sql
' OR 1=1 -- -
```

When injected into a standard login query, the raw query becomes:

```sql
SELECT * FROM logins WHERE username='tom' AND password='' OR 1=1;
````

Due to operator precedence, the database evaluates the condition with implicit grouping:

```sql
WHERE (username='tom' AND password='') OR (1=1);
```

### Evaluation Breakdown

1. **`AND` condition:** `(username='tom' AND password='')` evaluates to `FALSE` (assuming the password is incorrect or empty).
2. **`OR` condition:** `1=1` evaluates to `TRUE`.
3. **Final Result:** `FALSE OR TRUE` evaluates to `TRUE`.

Because the entire `WHERE` clause evaluates to `TRUE`, the database returns **all rows** in the table rather than filtering for a specific user.

### Why 'admin' is Returned

When an authentication query matches every row in the database:
- Web applications often fetch only the first record returned by the database driver (e.g., `LIMIT 1` or `fetch_assoc()`).
- Database tables typically store the initial administrator account in the very first row (ID 1).
- As a result, a generic `OR 1=1` payload logs the attacker in as the first user in the table (`admin`), rather than the targeted user (`tom`).

---

```ad-note
Conditions inside parenthesis `()` are evaluated before other conditions.
```

---

# Union Injections

## Detect number of columns

We can use the `ORDER BY` or `UNION` to determine the number of the columns.
### Using ORDER BY

We can start with `ORDER BY 1`, then `ORDER BY 2`, etc. until we reach a number that returns an error.

```ad-example
If we failed at `ORDER BY 4`, that means we have 3 columns.
```

