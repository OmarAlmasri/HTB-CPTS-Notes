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

# Bypassing Basic Authentication

Exploiting HTTP Verb Tampering is a straightforward process, we just need to alternate HTTP methods to see how they are handled by the web server and the web app.

## Identify

![[FileManager.png]]

In this File Manager application example, if we tried to use the Reset function to delete all files, it will ask for authentication using an HTTP Basic Auth prompt.

![[FileManager Auth Prompt.png]]

And we don't have credentials, that's why we will get a `401 Unauthorized` page.

## Exploit

To try and exploit the page, we need to identify the HTTP request method used  by the web app. We can intercept the request and examine it

![[FileManager Exploitation - 1.png]]

We can see the page is fetched via `GET` request, if we send a `POST` request and see whether the web page allows `POST` requests. We will get the same Basic Auth Prompt.

To see whether the server accepts `HEAD` requests, we can send an `OPTIONS` request to it and see what HTTP methods are accepted, as follows:

```sh
curl -i -X OPTIONS http://SERVER_IP:PORT/ 

HTTP/1.1 200 OK 
Date: 
Server: Apache/2.4.41 (Ubuntu) 
Allow: POST,OPTIONS,HEAD,GET 
Content-Length: 0 
Content-Type: httpd/unix-directory
```

The response shows `Allow: POST,OPTIONS,HEAD,GET`

If we tried the `HEAD` method, we will get an empty page, but now if we return to the home page, we will see that all files are successfully delted.

# Bypassing Security Filters

This type of HTTP Verb Tampering is caused by **Insecure Coding** errors made during the development of the application which lead to web app not covering all HTTP methods in certain functions.
## Identify

In the File Manager example, if we tried creating a new file with special characters in its name (e.g. `test;`), we will get an error message stating that a `Malicious Request Denied`.

This message indicates that the web app uses certain filters on the back-end to identify injection attempts. However, we can try HTTP Verb Tampering to see if we can bypass the security filter.
## Exploit

If we tried our injection payload and changed the request method from `GET` to `POST` request

![[FileManager Exploitation - 2.png]]

After sending the previous request, the payload will be executed successfully.

# Verb Tampering Protection

## Insecure Configuration

HTTP Verb Tampering can occur in most modern web servers, including `Apache`, `Tomcat`, and `ASP.NET`. It happens when we limit a page's authorization to a particular set of HTTP verbs/methods, which leaves the other remaining methods unprotected.

```xml title=".htaccess insecure configuration"
<Directory "/var/www/html/admin"> 
	AuthType Basic 
	AuthName "Admin Panel" 
	AuthUserFile /etc/apache2/.htpasswd 
	<Limit GET> 
		Require valid-user 
	</Limit> 
</Directory>
```

And the following example shows the same vulnerability for a **Tomcat** webserver

```xml title="web.xml insecure example"
<security-constraint> 
	<web-resource-collection> 
		<url-pattern>/admin/*</url-pattern> 
		<http-method>GET</http-method> 
	</web-resource-collection> 
	<auth-constraint> 
		<role-name>admin</role-name> 
	</auth-constraint> 
</security-constraint>
```

Finally, the following is an example for an `ASP.NET` configuration

```xml title="web.config insecure example"
<system.web> 
	<authorization> 
		<allow verbs="GET" roles="admin"> 
		<deny verbs="GET" users="*"> 
	</deny> 
	</allow> 
	</authorization> 
</system.web>
```

We can see that the authorization is being limited only to the `GET` method.
## Insecure Coding

