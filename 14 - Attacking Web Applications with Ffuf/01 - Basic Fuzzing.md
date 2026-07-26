# Recursive Fuzzing

In `ffuf`, we can enable recursive scanning with the `-recursion` flag, and we can specify the depth with the `-recursion-depth` flag.

```sh
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

