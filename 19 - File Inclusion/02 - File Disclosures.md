# # Local File Inclusion (LFI)
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

