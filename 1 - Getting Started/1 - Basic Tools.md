# Using Tmux

The default key to input `tmux` commands prefix is `[CTRL + B]`. In order to open a new window in `tmux`, we can hit the prefix 'i.e. `[CTRL + B]`' and then hit `C`.

We see the numbered windows at the bottom. We can switch to each window by hitting the prefix and then inputting the window number, like `0` or `1`. We can also split a window vertically into panes by hitting the prefix and then `[SHIFT + %]`.

We can also split into horizontal panes by hitting the prefix and then `[SHIFT + "]`.

**Cheat Sheet:** https://tmuxcheatsheet.com/

| **tmux**         |                                |
| ---------------- | ------------------------------ |
| `tmux`           | Start tmux                     |
| `ctrl+b`         | tmux: default prefix           |
| `prefix c`       | tmux: new window               |
| `prefix 1`       | tmux: switch to window (`1`)   |
| `prefix shift+%` | tmux: split pane vertically    |
| `prefix shift+"` | tmux: split pane horizontally  |
| `prefix ->`      | tmux: switch to the right pane |

# Using Vim

Once we are finished editing a file, we can hit the escape key `esc` to get out of `insert mode`, back into `normal mode`. When we are in `normal mode`, we can use the following keys to perform some useful shortcuts:

| **Command** | **Description** |
| ----------- | --------------- |
| `x`         | Cut character   |
| `dw`        | Cut word        |
| `dd`        | Cut full line   |
| `yw`        | Copy word       |
| `yy`        | Copy full line  |
| `p`         | Paste           |
If we want to save a file or quit `Vim`, we have to press`:` to go into `command mode`.

There are many commands available to us. The following are some of them:

| **Command** | **Description**      |
| ----------- | -------------------- |
| `:1`        | Go to line number 1. |
| `:w`        | Write the file, save |
| `:q`        | Quit                 |
| `:q!`       | Quit without saving  |
| `:wq`       | Write and quit       |
**Cheat Sheet:**  https://vimsheet.com/

| **Vim**    |                                 |
| ---------- | ------------------------------- |
| `vim file` | vim: open `file` with vim       |
| `esc+i`    | vim: enter `insert` mode        |
| `esc`      | vim: back to `normal` mode      |
| `x`        | vim: Cut character              |
| `dw`       | vim: Cut word                   |
| `dd`       | vim: Cut full line              |
| `yw`       | vim: Copy word                  |
| `yy`       | vim: Copy full line             |
| `p`        | vim: Paste                      |
| `:1`       | vim: Go to line number 1.       |
| `:w`       | vim: Write the file 'i.e. save' |
| `:q`       | vim: Quit                       |
| `:q!`      | vim: Quit without saving        |
| `:wq`      | vim: Write and quit             |