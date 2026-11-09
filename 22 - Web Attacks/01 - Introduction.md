# Web Attacks
## HTTP Verb Tampering
An HTTP Verb Tampering attack exploits web servers that accept many HTTP verbs and methods. This can be exploited by sending malicious requests using unexpected methods, which may lead to bypassing the web application's authorization mechanisms or even bypassing its security controls against other web applications.
## Insecure Direct Object References (IDOR)
Very common, we can access other users' files and information by simply guessing or calculating their File/User IDs.
## XML External Entity (XXE) Injection

If a user was able to supply an XML input and the server processed this XML data, we might be able to disclose local files stored on the backend server. These files can 