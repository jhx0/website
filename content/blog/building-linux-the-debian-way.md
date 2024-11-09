+++
title = "Building Linux the Debian Way"
date = "2024-11-09T15:14:28+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["linux", "kernel", "debian", "package", "deb"]
+++

Sometimes it is helpful to build a **custom kernel**.   
Someone might want to **slim down** the kernel, **optimize** it or just building a **kernel** for fun - depending on ones needs of course.   
**Debian** makes this a **breeze**!.   
The following short article will show how to **build a Linux kernel** and also generating easy to install **deb packages**.

# Getting the kernel sources
The first step is always getting the **Linux kernel sources**.   
**Debian** has already **packaged** the **kernel sources** which can be easily installed via **apt**.

```shell
$ sudo apt install linux-source-6.9
```
**Note:** Install the **kernel** **version** that you need and want!

Another way of getting the **kernel sources** is **downloading** named **sources** directly from the **official site**, kernel.org.

These can be found here: [Kernel Sources](https://kernel.org)

If you've gone the route of leveraging **apt** you can list the contents of the package you have just installed via **dpkg**.

```shell
$ dpkg -L linux-source-6.9 
/.
/usr
/usr/share
/usr/share/doc
/usr/share/doc/linux-source-6.9
/usr/share/doc/linux-source-6.9/changelog.Debian.gz
/usr/share/doc/linux-source-6.9/copyright
/usr/src
/usr/src/linux-source-6.9.tar.xz
```

As you can see, the **sources** are stored under */usr/src/*.

# Preparing the build directory
I personally always **create a build directory** in my home folder.
For example:
```shell
$ mkdir build 
$ tar xJf /usr/src/linux-source-6.9.tar.xz -C build/
$ cd build/linux-source-6.9/
```
**Note:** Where you store/build your **kernel** is up to you.

# Installing needed dependencies
**Debian** makes this easy for us.

```shell
$ sudo apt install build-essential linux-source-6.9 bc kmod cpio flex libncurses5-dev libelf-dev libssl-dev dwarves bison fakeroot
$ sudo apt-get build-dep linux-source-6.9
```

After everything is installed we can move on to **configuring the kernel**.

# Configuration
First things first, **clean the kernel source tree**.
```shell
$ make mrproper
```

You could from this point on go straight to **configuring** the **kernel** via **menuconfig**/**nconfig**...   
But, given the fact that **Debian** ships the **kernel config** used for your **kernel**, you can make your life a little easier by using said **config**.   
In the following example I will **copy over the config** from a **installed kernel** so that I already have most things taken care of.

```shell
$ cp -v /boot/config-6.9.10+bpo-amd64 .config
```

The next step will be to actually dive into **configuring the kernel** via a **friendly curses menu**.

```shell
$ make nconfig
```

**Note:** Make sure to **save your configuration changes**! The name of the resulting config will be **.config** (Directly in the **kernel source directory**)

# Make the CPU go brrrrrr
**All set** - Time to **build the kernel** and **generate** easy to install **deb packages**.

```shell
$ make -j`nproc` bindeb-pkg
```
**Note:** Leveraging *"nproc"* will return **all usable CPU cores**. Be careful with this since it will use your **CPU** to it's **full extent**!

# Final chapter
The **kernel was built**. A big step is taken care of.   
You will find the **generated deb packages** in the **build directory** we've created before.

```shell
$ cd ..
$ ls
total 1013080
-rw-r--r-- 1 x x   9451452 Nov  9 15:13 linux-headers-6.9.10_6.9.10-1_amd64.deb
-rw-r--r-- 1 x x 100806300 Nov  9 15:13 linux-image-6.9.10_6.9.10-1_amd64.deb
-rw-r--r-- 1 x x 925761052 Nov  9 15:14 linux-image-6.9.10-dbg_6.9.10-1_amd64.deb
-rw-r--r-- 1 x x   1356640 Nov  9 15:12 linux-libc-dev_6.9.10-1_amd64.deb
drwxr-xr-x 1 x x      2166 Nov  9 15:12 linux-source-6.9
-rw-rw-r-- 1 x x      6461 Nov  9 15:14 linux-upstream_6.9.10-1_amd64.buildinfo
-rw-rw-r-- 1 x x      2191 Nov  9 15:15 linux-upstream_6.9.10-1_amd64.changes
```

**Q:** So, what is left to do?   
**A:** Of course, **installing the packages** so we can **boot into our newly baked kernel**.

**apt**:
```shell
$ sudo apt install ./*.deb
```
**dpkg**:
```shell
$ sudo dpkg -i *.deb
```
Select which better suits **your needs**.

# The final countdown
All there is left to do is to **reboot the system** and **selecting** the newly compiled kernel during the **Grub boot menu** - if you are using **Grub** that is.   
That is literally it - **We are done**.

# Closing words
This is merely one way to do it... and yes, you guessed it... there is **more than one way to do it**.   
You could also entirely leave out **Debian** specific parts and **compile a kernel** from scratch without **generating** any **deb packages** at all.   

In the end: **You have to chose what fits you best**.   

Lust but not least...

**Stay Open!**