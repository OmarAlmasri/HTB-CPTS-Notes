# SQLMap Output Description
## Log Messages Description
#### URL content is stable

```sh
- "target URL content is stable"
```

Means there are no major changes between responses.

#### Parameter appears to be dynamic

```sh
- "GET parameter 'id' appears to be dynamic"
```

It is always desired to test parameters that appear to be dynamic as it signs that any change to its value would result in a change in the response; hence the parameter may be linked to a database.

#### Parameter might be injectable

```sh
- "heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')"
```

DBMS errors are good indicators of the potential SQLi. In this case, there was a MySQL error when `sqlmap` sends intentionally invalid value.

#### Parameter might be vulnerable to XSS attacks

```sh
- "heuristic (XSS) test shows that GET parameter 'id' might be vulnerable to cross-site scripting (XSS) attacks"
```

While it is not its primary purpose, SQLMap also runs a quick heuristic test for the presence of an XSS vulnerability.

#### Back-end DBMS is '...'

```sh
- "it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? Y/n"
```

If SQLMap found a clear indication about the DBMS type, we can narrow down the payloads to that specific DBMS.

#### Level/risk values

```sh
- "for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? Y/n"
```

If there is a clear indication that the target uses the specific DBMS, it is also possible to extend the tests for that same specific DBMS beyond the regular tests.

#### Reflective values found

```sh
- "reflective value(s) found and filtering out"
```

Just a warning that parts of the used payloads are found in the response.

#### Parameter appears to be injectable

```sh
- "GET parameter 'id' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable (with --string="luther")"
```

This message indicates that the parameter appears to be injectable, though there is still a chance for it to be a false-positive finding.

#### Extending UNION query injection technique tests

```sh
- "automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found"
```

#### Technique appears to be usable

```sh
- "ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test"
```

As a heuristic check for the UNION-query SQLi type, before the actual `UNION` payloads are sent, a technique known as `ORDER BY` is checked for usability.

#### Parameter is vulnerable

```sh
- "GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? y/N"
```

Most important message, it means that the parameter was found to be vulnerable to SQL Injections.

#### Sqlmap identified injection points

```sh
- "sqlmap identified the following injection point(s) with a total of 46 HTTP(s) requests:"
```

Following after is a listing of all injection points with type, title, and payloads, which represents the final proof of successful detection and exploitation of found SQLi vulnerabilities. It should be noted that SQLMap lists only those findings which are provably exploitable (i.e., usable).

