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

