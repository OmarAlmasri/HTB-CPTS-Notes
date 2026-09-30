# Client-Side Validation

Some web applications rely only on Frontend side validation for types of uploaded files.
Like opening the upload dialog with only images extensions showing. This kind of validation can be easily bypassed with changing the image only filter to `All types` or just edit the code in the client side.

That's why all validations must be done in the server-side.

## Back-end Request Modification

We can catch a normal upload request using BurpSuite and modify the extension of the file from the intercepted request.

```ad-info
We may also modify the `Content-Type` of the uploaded file
```

## Disabling Front-end Validation

Another method to bypass client-side validations is through manipulating the front-end code. As these functions are being completely processed within our web browser, we have complete control over them.

To start, we can open `Page Inspector`, click on the place which is where we trigger the file selector for the upload form:

![[Disable Front-end validation via Page Inspector.png]]


---

# Blacklist Filters

**Blacklisted Extensions Example:**

```php
$fileName = basename($_FILES["uploadFile"]["name"]);
$extension = pathinfo($fileName, PATHINFO_EXTENSION);
$blacklist = array('php', 'php7', 'phps');

if (in_array($extension, $blacklist)) {
    echo "File type not allowed";
    die();
}
```

We can utilize different type of extensions to bypass this filter and still be able to execute PHP code on the backend server. Also, the validation used in the function is case-sensitive, so we can upload a PHP file with mixed-case extension (e.g. `.pHp`).

## Fuzzing Extensions

We should fuzz the file upload functionality with a wordlist of different file extensions and look if any extension succeeds. 

```ad-resources
There are many lists of extensions we can use in our fuzzing scan. **PayloadAllTheThings** provides list of extensions for [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) and [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) web applications. We may also use **SecLists** list of common Web Extensions.
```


---

# Whitelist Filters

**Whitelisted Extensions Example:**

```php
$fileName = basename($_FILES["uploadFile"]["name"]); if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) { echo "Only images are allowed"; die(); }
```

There are different techniques to bypass it, let's see how:
## Double Extensions

The code above only tests whether the file name contains an image extension; we can bypass this regex via **Double Extensions**.

**Example:** `shell.jpg.php`

However, this might not work as some web apps may use a strict `regex` pattern

```php
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) { ...SNIP... }
```

This pattern only consider the final extension, so the above attack will not work.

## Reverse Double Extension

In some cases, the web app might not be vulnerable but the web server configuration may lead to a vulnerability.

For example, the `/etc/apache2/mods-enabled/php7.4.conf` for the `Apache2` web server may include the following configuration:

```xml
<FilesMatch ".+\.ph(ar|p|tml)">     
	SetHandler application/x-httpd-php 
</FilesMatch>
```

As we can see, the regex used is vulnerable and double extension file can be used to execute PHP code with a file such as `shell.php.jpg` as it includes `.php` in its name.

## Character Injection

We can inject several characters before or after the final extension to cause a web application to misinterpret the filename and execute the uploaded file as a PHP script.

The following are some of the characters we may try injecting:

- `%20`
- `%0a`
- `%00`
- `%0d0a`
- `/`
- `.\`
- `.`
- `…`
- `:`


---

# Type Filters

Our past attacks dealt only with file extensions in the file name.
Modern web applications and servers also test the content of the uploaded file to ensure it matches the specified type. While extension filters accepts several extensions, content filters usually specify a single category (e.g. images, videos, documents), which is why they don't typically use blacklists or whitelists.

There are two common methods for validating the file content: `Content-Type Header` or `File Content`

## Content-Type

The following is an example of how a PHP web application tests the Content-Type header to validate the file type:

```php
$type = $_FILES['uploadFile']['type']; 

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) 
{ 
	echo "Only images are allowed"; 
	die(); 
}
```

We may start by fuzzing the `Content-Type` header with the SecList's [SecLists/Discovery/Web-Content/web-all-content-types.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt) wordlist through burp intruder.

We can filter the content of the wordlist to only have image-related Content Types

```sh
cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

```ad-note
A file upload HTTP request has two Content-Type headers, one for the attached file (at the bottom), and one for the full request (at the top). We usually need to modify the file's Content-Type header, but in some cases the request will only contain the main Content-Type header (e.g. if the uploaded content was sent as `POST` data), in which case we will need to modify the main Content-Type header.
```

## MIME-Type

**MIME-Type** is an internet standard that determines the type of a file through its general format and bytes structure.

This is usually done by inspecting the first few bytes of file's content, which contain the [File Signature](https://en.wikipedia.org/wiki/List_of_file_signatures) or [Magic Bytes](https://web.archive.org/web/20240522030920/https://opensource.apple.com/source/file/file-23/file/magic/magic.mime). *(E.g. if a file starts with GIF87a or GIF89a this indicates it's a GIF image)*.
#### Find the file type on Linux through MIME type

```sh
echo "This is a text file" > text.jpg
file text.jpg

text.jpg: ASCII text
```

```sh
echo "GIF8" > text.jpg
file text.jpg

text.jpg: GIF image data
```

Web servers also use this standard to determine file types, which is more accurate than testing the file extension.

**Example Code:**

```php
$type = mime_content_type($_FILES['uploadFile']['tmp_name']); 

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) 
	{ 
		echo "Only images are allowed"; die(); 
	}
```

```ad-tip
We can use a combination of the two methods discussed in this section, which may help us bypass some more robust content filters. For example, we can try using an `Allowed MIME type with a disallowed Content-Type`, an `Allowed MIME/Content-Type with a disallowed extension`, or a `Disallowed MIME/Content-Type with an allowed extension` and so on similarly.
```

