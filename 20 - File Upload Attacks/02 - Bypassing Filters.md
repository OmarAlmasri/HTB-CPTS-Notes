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

