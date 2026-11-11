# Identifying IDORs
## URL Parameters & APIs

The very first step in exploiting IDORs is identifying Direct Object References. We should study the HTTP requests to look for URL parameters or APIs with an object reference (e.g. `?uid=1` or `?filename=file_1.pdf`). These are mostly found in URL parameters or APIs but may also be found in other HTTP headers, like cookies.

## AJAX Calls

```ad-tldr
Check JavaScript code in the front-end and try to identify any AJAX calls to specific end-points or APIs that contain direct object references.
```

The following example shows a basic example of an AJAX call:

```js
function changeUserPassword() { $.ajax({ url:"change_password.php", type: "post", dataType: "json", data: {uid: user.uid, password: user.password, is_admin: is_admin}, success:function(result){ // } }); }
```

## Understand Hashing/Encoding