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