# Upgrading TTY

```sh
python -c 'import pty; pty.spawn("/bin/bash")'
```

```shell
www-data@remotehost$ ^Z

Ripcord88x@htb[/htb]$ stty raw -echo
Ripcord88x@htb[/htb]$ fg

[Enter]
[Enter]
www-data@remotehost$
```

We may notice that our shell does not cover the entire terminal. To fix this, we need to figure out a few variables. We can open another terminal window on our system, maximize the windows or use any size we want, and then input the following commands to get our variables:


```shell
Ripcord88x@htb[/htb]$ echo $TERM

xterm-256color
```


```shell
Ripcord88x@htb[/htb]$ stty size

67 318
```

The first command showed us the `TERM` variable, and the second shows us the values for `rows` and `columns`, respectively. Now that we have our variables, we can go back to our `netcat` shell and use the following command to correct them:

```shell
www-data@remotehost$ export TERM=xterm-256color

www-data@remotehost$ stty rows 67 columns 318
```

Once we do that, we should have a `netcat` shell that uses the terminal's full features, just like an SSH connection.

# Web Shells

| **Web Server** | **Default Webroot**    |
| -------------- | ---------------------- |
| `Apache`       | /var/www/html/         |
| `Nginx`        | /usr/local/nginx/html/ |
| `IIS`          | c:\inetpub\wwwroot\|   |
| `XAMPP`        | C:\xampp\htdocs\|      |
