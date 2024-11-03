+++
title = "Generating Passwords"
date = "2024-11-03T14:07:41+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["linux", "bsd", "debian", "ubuntu", "freebsd", "diceware", "xkcdpass", "openssl", "random", "password", "generator", "cli", "shell"]
+++

# Password Generators
Imagine needing to come up with **passwords** all the time, which is not impossible.   
However, why not leverage tools for doing that. After all, generating **random passwords** is a important step.    
Let's explore some of the options out there and see what can be done.

# pwgen
One of my favorites, period. Using **pwgen** is easy and the tool itself is available everywhere.   

**Installation:**
```shell
$ sudo apt install pwgen
$ sudo pacman -S pwgen
$ sudo pkg install pwgen
```

Now, let's look at a **example**:
```shell
$ pwgen 32 1
```
This will generate one passwort with a length of **32 characters**. Easy.   
**pwgen** can do more than that though.
```shell
$ pwgen -cyn 32 1
```
**Explanation:**
- Include at least one capital letter (-c)
- Include at least one special character (-y)
- Include at least one number (-n)
- Generate one password of length 32 characters

# Diceware
**Diceware** is antoher very useful utility to generate so called **passphrases**. A **passphrase** is something one can actually remember - instead of glueing random characters together.

**Installation (Not packaged for FreeBSD):**
```shell
$ sudo apt install diceware
$ sudo yay -S diceware (AUR)
```
**Following, a example:**
```shell
$ diceware --no-caps -d " " -n 4
```
**Explanation:**
- No capitalized characters (--no-caps)
- Delimiter (Space)
- Number of words used (-n 4)

**Example Output:**
```shell
magician vice goliath diagram
```
As you can for sure tell, the generated **passphrase** is quite easy to commit to memory.

# xkcdpass
Another quit useful tool for generating **passphrases** is **xkcdpass** which is easy to use.

Installation:
```shell
$ sudo apt install xkcdpass
$ sudo pacman -S xkcdpass
$ sudo pkg install py311-xkcdpass
```
**Example usage:**
```shell
$ xkcdpass
```

**Example Output:**
```shell
taunt playmaker aviation shout unbent tiara
```

Same as **diceware**, the generated **passphrases** are easier to remember than having long strings of random characters/numbers.

# Shell magic (Bonus)
One can also generate **passwords** without using external tools.   
The following command generates a 32 character **password** leveraging just builtin commands.
```shell
$ tr -dc A-Za-z0-9 </dev/urandom | head -c 32; echo
```

**Another example:**
```shell
$ cat /dev/urandom | env LC_CTYPE=C tr -dc a-zA-Z0-9 | head -c 32; echo
```

# Using OpenSSL
Another option is using the **openssl** commandline utility. Since the command **openssl** is available everywhere, this will give some flexibility - if nothing else is at hand.

```shell
$ openssl rand -base64 32
$ openssl rand -hex 32
```
**Explanation:**
- The first call to openssl generates a random Base64 value
- The second call to openssl does the same except it uses Hexadecimal

# Yet antoher way
The **quickest and cheapest** way to generate a **password** in my opinion would be to just calculate a **hash** from a file for example and using this very **hash** as a **password**.

```shell
$ sha1sum /etc/fstab
```

**Output:**
```shell
a0667f7cc12164bcca8fc0ca3f9492537818185a  /etc/fstab
```

Although, using a proper **password** generator is still the best option if you ask me.

# Closing words
There are multiple ways one can go to generate **passwords** - And I'm sure that there are many more ways.   
The example/commands given here serve as a example on how flexible different commands are.   
As always, everyone has there own favorit or way to do it.

Hope you enjoyed the article.

**Stay Open!**