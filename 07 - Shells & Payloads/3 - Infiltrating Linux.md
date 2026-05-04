# Spawning Interactive Shells

```sh
/bin/bash -i
```

```perl
perl -e 'exec "/bin/sh";'
```

```perl
perl: exec "/bin/sh";
```

```ruby
ruby: exec "/bin/bash"
```

```lua
lua: os.execute('/bin/bash')
```

```sh title="AWK to Shell"
awk 'BEGIN {system("/bin/bash")}'
```

```sh
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

```sh
find . -exec /bin/sh \; -quit
```

## Vim

**Vim to Shell:**

```sh
vim -c ':!/bin/sh'
```

**Vim Escape:**

```sh
vim :set shell=/bin/sh :shell
```

