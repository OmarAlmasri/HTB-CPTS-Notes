# Intro to HTTP Verb Tampering

If a server was configured to only accept `GET`/`POST` requests and we send a request with another method such as `HEAD` or `PUT`, an error page might be displayed which could lead to an information disclosure. On the other hand, if the web server configs are not restricted to only accept the HTTP methods required by the web server (e.g. `GET`/`POST`), and the web app was not developed to handle other types of HTTP requests, we may be able to exploit this insecure configs to gain access to functionalities we do not have access to or bypass certain security controls.
## HTTP Verb Tampering

Other than `GET` and `POST`, the following are some of the commonly used HTTP verbs:

| Verb      | Description                                                                                         |
| --------- | --------------------------------------------------------------------------------------------------- |
| `HEAD`    | Identical to a GET request, but its response only contains the `headers`, without the response body |
| `PUT`     | Writes the request payload to the specified location                                                |
| `DELETE`  | Deletes the resource at the specified location                                                      |
| `OPTIONS` | Shows different options accepted by a web server, like accepted HTTP verbs                          |
| `PATCH`   | Apply partial modifications to the resource at the specified location                               |
 If a web server is not securely configured to manage these methods, we can use them to gain control over the back-end server.

## Insecure Configurations
If a web server's auth config was limited to specific HTTP methods, this would leave some HTTP methods accessible without authentication.

```xml title="Insecure Configuration Example"
<Limit GET POST> Require valid-user </Limit>
```

## Insecure Coding

This can occur when a web developer applies specific filters to mitigate particular vulnerabilities while not covering all HTTP methods with that filter.

```php title="Insecure Coding Example"
$pattern = "/^[A-Za-z\s]+$/"; 

if(preg_match($pattern, $_GET["code"])) { 
	$query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'"; 
	...SNIP... 
}
```

