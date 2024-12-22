+++
title = "Building and Patching Vsftpd"
date = "2024-12-22T15:28:08+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["debian", "linux", "vsftpd", "compiling", "building", "deb", "apt", "package"]
+++

In this little **guide** we will create a custom **.deb package** from **vsftpd** with the version information **patched** out.

**Disclaimer:** Changing the version number is **NOT** magically saving you from everything! **Security** is a **layered approach** and is compromised of many steps - not just one! (**Keep that in mind**)

# Getting the sources
**First things first**. We need to get the current **sources** from the **Debian repository** with the following command.
```shell
$ apt-get source vsftpd
```

The **sources** will be automatically unpacked by **apt-get** and we can continue to alter the **vsftpd** **source code**.

# Gimme the source!
After the **sources** have been unpacked we can inspect the code to find out where the **version number** is noted.

```shell
$ grep -ri VERSION *.h
```

Which will produce the following **output**:
```shell
[chaos ~/Work/vsftpd-3.0.3] $ grep -ri VERSION *.h
sysdeputil.h:/* Or a softer version delivering SIGTERM. */
vsftpver.h:#ifndef VSF_VERSION_H
vsftpver.h:#define VSF_VERSION_H
vsftpver.h:#define VSF_VERSION "3.0.3"
vsftpver.h:#endif /* VSF_VERSION_H */
```
We can see that the **define constant** **VSF_VERSION** holds the current **version** number for **vsftpd**.   
In the next step we will **change this very constant** to output a different **version string**.

# Altering the source
Let's open the **header** file and change the version number.
```shell
$ vim vsftpd-3.0.3/vsftpver.h
```
For my purpose, I will simply **change** the version string to **"unknown"**.

```shell
...
#define VSF_VERSION "unknown"
...
```

# Dependencies
Before we begin to make a **.deb package** we need to get the needed **build dependencies** - which are necessary to actually do the **compiling**.

```shell
$ sudo apt build-dep vsftpd
```

If you want to **list the dependencies** outside of getting the needed packages you can do so with **apt-cache**.
```shell
$ apt-cache showsrc vsftpd | grep ^Build-Depends
Build-Depends: debhelper (>= 11), libcap2-dev [linux-any], libpam0g-dev, libssl-dev, libwrap0-dev
```

# Making a .deb package
**Debian** makes this very easy and comfortable for us. All we need to do is leverage **dpkg-buildpackage** to compile and make the **package**.

```shell
$ dpkg-buildpackage -b
```

If all went well we get the following output.
```shell
...
dpkg-deb: building package 'vsftpd' in '../vsftpd_3.0.3-13_amd64.deb'.
 dpkg-genbuildinfo --build=binary -O../vsftpd_3.0.3-13_amd64.buildinfo
 dpkg-genchanges --build=binary -O../vsftpd_3.0.3-13_amd64.changes
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)
```

**That's it** - Literally.

# It's apt time
The only thing to do is **installing** the newly created **.deb package** and enjoying the fruits of our labor.
```shell
$ sudo apt install ./vsftpd_3.0.3-13_amd64.deb
```
**Hint:** The version might be different for your depending on when you follow along this tutorial!

# Probing vsftpd
We got the **.deb package** build and we installed it, great!   
But how can we make sure that it **actually worked**?   
**Simple**, first we make sure **vsftpd** is running.

```shell
$ sudo systemctl (re)start vsftpd
```

So far so good... but can we see the **version number** somehow?   
Introducing **nmap**. A simple **nmap scan** will tell us the version of **vsftpd** (**Banner grabbing**) when the right switches were passed to it as follows.

```shell
$ sudo nmap -sS -sV localhost -p21
```
**Hint:** **-sS** initiates a **stealth scan** and **-sV** probes for the **version**. **-p21** tells nmap to just scan **TCP port 21**.

We now get the following output from our **nmap scan**.
```shell
[chaos ~/Work] $ sudo nmap -sS -sV localhost -p21
Starting Nmap 7.93 ( https://nmap.org ) at 2024-12-22 16:11 CET
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000036s latency).
Other addresses for localhost (not scanned): ::1

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd unknown
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.22 seconds
```

As we can see, the reported version is now **"unknown"** - the **custom** version/string we've set before!

# Final words
Of course, we need to address the **elephant in the room**:
This is **NOT** the only thing one has to undertake to secure a system/service!   
**I can only repeat that**, do not falsly believe that by just changing the reported version number of a program that you are now somehow free from all trouble.
**Security is a process** and needs much more work than what we've done just here.   
It is however a nice added bonus to our defense - by making it harder for a attacker to evaluate the version used for a given service.

Be aware however that **apt** will happily install a new version of **vsftpd** over the one you've just installed from a **.deb package**.   
You could leverage **apt-mark hold** to make sure the package does not get overwritten with a newer version - but this is left as a excercise to the reader.

**Stay Open everyone!**

...And to all of you: **Merry Xmas in advance!**