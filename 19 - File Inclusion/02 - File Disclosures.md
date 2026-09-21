# Local File Inclusion (LFI)
## Basic LFI

![[File Inclusion.png]]

## Path Traversal

This code examples uses **Absolute Path**

```php
include($_GET['language']);
```

so the payload would be `/etc/passwd`

While this example, the `languages` directory is used as a **Relative Path**, so the payload should include path traversal 

```php
include("./languages/" . $_GET['language']);
```

![[Path Traversal.png]]

## Filename Prefix

On some occasions, our input may be appended after a different string. For example, it may be used with a prefix to get the full filename, like the following example:

```php
include("lang_" . $_GET['language']);
```

In this case, if we try to traverse the directory with `../../../etc/passwd`, the final string would be `lang_../../../etc/passwd`, which is invalid:

![[Filename Prefix Error.png]]

So instead of directly using path traversal, we can prefix a `/` before our payload, and this should consider the prefix as a directory:

![[Filename Prefix Bypass.png]]

## Appended Extensions

Another very common example is when an extension is appended to the `language` parameter, as follows:

```php
include($_GET['language'] . ".php");
```

## Second-Order Attacks

```ad-example
Like when you use an LFI paylaod in your username and it gets interpreted as a real file path and you get the content of the file when you view your username.
```


---

# Basic Bypasses

## Non-Recursive Path Traversal Filters

One of the most basic filters against LFI is a search and replace filter, where it simply deletes substrings of `../` to avoid path traversals:

```php
$language = str_replace('../', '', $_GET['language']);
```

This can be easily bypassed with payloads from the type:

```php
....//
# OR
..\/
```

## Encoding

Some filters removes out potential LFI characters such as: `.` or `/`
So you'll need to try different types of encoding like URL Encoding or Double URL Encoding.

## Approved Paths

Some web applications may also use Regular Expressions to ensure that the file being included is under a specific path.

**Example:** This application only accepts paths that are under the `./languages` directory, as follows:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

![[Approved Paths.png]]

## Appended Extension

The following techniques are obsolete with modern versions of PHP and only work with PHP versions before **5.3/5.4**
#### Path Truncation

In earlier versions of PHP, defined strings have a max length of 4096 characters. If a longer string is passed, it will be **truncated**, and any characters after the max length will be ignored.

We could write a very long string that when it reaches the 4096 characters limitation, the appended extension `.php` would be truncated.

**Example Payload:**

```php
?language=non_existing_directory/../../../etc/passwd/./././././ REPEATED ~2048 times]
```

#### Null Bytes

PHP versions before **5.5** where vulnerable to **null byte injections**, which means that adding a null byte (`%00`) at the end of the string would terminate the string and not consider anything after it.

---

# PHP Filters

```ad-resources
If we found a website vulnerable to LFI, we can use **PHP Filters** to extend our LFI capablities and potentially get an RCE

[PHP: php:// - Manual](https://www.php.net/manual/en/wrappers.php.php)
```

## Input Filters

[PHP Filters](https://www.php.net/manual/en/filters.php) allow us to transform stream data by applying specific filters during stream operations.

To use **PHP Filters** we can use schemes such as `php://`, and we can access the PHP filter wrapper with `php://filter/` to apply filters to a resource.

The **filter** wrapper has several parameters, the mainly ones we use for our attacks are `resource` and `read`.

The **resource** parameter is required for filter wrappers, and with it we can specify the stream we would like to apply the filter on (e.g. local file). While the **read** parameter can apply different filters on the input resource, so we can use it to specify which filter we want to apply on our resource.

In short:

- **`resource`**: Points to the file you want to read.
- **`read`**: Specifies the filter (the action) to apply to that file, like Base64 encoding.

By using them together, you can trick the server into giving you the source code of a PHP file instead of executing it.

```ad-resources
There are four different types of filters available for use, which are [String Filters](https://www.php.net/manual/en/filters.string.php), [Conversion Filters](https://www.php.net/manual/en/filters.convert.php), [Compression Filters](https://www.php.net/manual/en/filters.compression.php), and [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php).
```

The filter that is useful for LFI attacks is the `convert.base64-encode` filter, under **Conversion Filters**.

## Fuzzing for PHP Files

```sh
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```

## Standard PHP Inclusion

```html
http://<SERVER_IP>:<PORT>/index.php?language=config
```

This might result in disclosing `config.php` file to us.

## Source Code Disclosure

Once we have a list of potential PHP files we want to read, we can start disclosing their source with the `base64` filter

```php
php://filter/read=convert.base64-encode/resource=config
```

