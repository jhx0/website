+++
title = "Creating a Ventoy Usb Stick"
date = "2024-11-09T16:44:51+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["ventoy", "usb", "boot", "linux", "windows", "bsd", "iso", "os"]
+++

**Ventoy** is a fantastic way of **booting many different iso's** on computers.   
I have used to **flash usb sticks** all the time with different **operating systems** - which in and of itself is fine... except one having many **usb sticks** that need to be updated and the dreaded situation in which said sticks are not labeled... we all know where that ends.   

To completly avoid that one can leverage **Ventoy**.   

The following is a short article on how to create a **Ventoy boot usb stick**.

# What do we need?
There are only **two things** needed:
- A **decently sized usb stick** (I recommend **16GB or more** - depending on how many iso's you want to store)
- **Ventoy** itself

Let's get started with getting the latest **Ventoy** version.

# Getting Ventoy
First, we have to get the newest **Ventoy** version from **Sourceforge (Project site)**.

```shell
$ firefox https://sourceforge.net/projects/ventoy/files/
```
**Note:** Be aware that there are **different options to chose from**. In our example we will download the **Linux archive**.

Next thing on the agenda is **extracting the archive**.

```shell
$ tar xzf ventoy-1.0.99-linux.tar.gz
```
**Note:** The **version number** might have **changed** when you read this very article!

After extracting we find the following **archive contents**.

```shell
[$ ls -lA
total 184
drwxr-xr-x 1 x x    38 Nov  9 16:49 boot
-rwxr-xr-x 1 x x  3111 Jun  8 11:19 CreatePersistentImg.sh
-rwxr-xr-x 1 x x  3022 Jun  8 11:19 ExtendPersistentImg.sh
drwxr-xr-x 1 x x    12 Nov  9 16:49 plugin
-rw-r--r-- 1 x x  2558 Jun  8 11:19 README
drwxr-xr-x 1 x x   298 Nov  9 16:49 tool
drwxr-xr-x 1 x x    92 Nov  9 16:49 ventoy
-rwxr-xr-x 1 x x  1778 Jun  8 11:19 Ventoy2Disk.sh
-rwxr-xr-x 1 x x 38456 Jun  8 11:19 VentoyGUI.aarch64
-rwxr-xr-x 1 x x 30592 Jun  8 11:19 VentoyGUI.i386
-rwxr-xr-x 1 x x 42944 Jun  8 11:19 VentoyGUI.mips64el
-rwxr-xr-x 1 x x 32704 Jun  8 11:19 VentoyGUI.x86_64
-rwxr-xr-x 1 x x  5389 Jun  8 11:19 VentoyPlugson.sh
-rwxr-xr-x 1 x x  7064 Jun  8 11:19 VentoyVlnk.sh
-rwxr-xr-x 1 x x  2971 Jun  8 11:19 VentoyWeb.sh
drwxr-xr-x 1 x x    54 Nov  9 16:49 WebUI
```

# Deploying Ventoy on a USB stick
There are **multiple files found in the extracted archive** but we are only interested in the **Ventoy2Disk.sh** script.

After you've **plugged in** your **usb stick** you need to check which **device name** it has been assigned.

```shell
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    1    59G  0 disk 
├─sda1        8:1    1    59G  0 part 
└─sda2        8:2    1    32M  0 part
...
```
As you can see I have a **64GB usb** stick plugged in which I will use for **Ventoy**.

Next, **change into the directory** where you've extracted **Ventoy**.
After that we can **proceed to the installation** part.

Follow the prompts and answer **"y"** to the questions asked.

**Warning:** This will **delete ALL data** on the plugged in usb stick!

```shell
$ sudo ./Ventoy2Disk.sh -i /dev/sda

**********************************************
      Ventoy: 1.0.99  x86_64
      longpanda admin@ventoy.net
      https://www.ventoy.net
**********************************************

Disk : /dev/sda
Model:  Patriot Memory (scsi)
Size : 59 GB
Style: MBR


Attention:
You will install Ventoy to /dev/sda.
All the data on the disk /dev/sda will be lost!!!

Continue? (y/n) y

All the data on the disk /dev/sda will be lost!!!
Double-check. Continue? (y/n) y

Create partitions on /dev/sda by parted in MBR style ...
Done
Wait for partitions ...
partition exist OK
create efi fat fs /dev/sda2 ...
mkfs.fat 4.2 (2021-01-31)
success
Wait for partitions ...
/dev/sda1 exist OK
/dev/sda2 exist OK
partition exist OK
Format partition 1 /dev/sda1 ...
mkexfatfs 1.3.0
Creating... done.
Flushing... done.
File system created successfully.
mkexfatfs success
writing data to disk ...
sync data ...
esp partition processing ...

Install Ventoy to /dev/sda successfully finished.
```

**That's it - Literally**.   
A **Ventoy** usb Stick was **just created** for you!

# What about ISO's?
After a successful deployment **Ventoy** has created a **partition** named **Ventoy** where you can simply move your **iso files** of choice to.

```shell
$ sudo fdisk -l
...
/dev/sda1  *         2048 123666431 123664384  59G  7 HPFS/NTFS/exFAT
...
```
The partition is formated with **exFAT** which also makes it easy to **copy over iso files** from a **Windows** system for example (Or any other system that supports **exFAT**).

You can access this very partition via your **GUI filemanager** of choice or **via the shell** (By **mounting** it to a directory of your choice).

# All your ISO belong to us
**We are all set**.   

There is nothing more to do than **copying over iso files**.   
Depending on your needs you might want to invest into a **decent usb stick** that has the needed **storage space** and is at least a little faster than some knockoff usb stick that can barely keep up.

You can from now on **insert the stick** into any **PC/Laptop/Whatever** and select the **Ventoy** stick in the **boot menu** (Depending on the **mainboard manufacturer**, **the key for booting varies**... Mostly **F8**, **F10** or **F12** though).

# Final words
I've used to **flash** many **usb sticks** during my lifetime, many.   
Truth is that sometimes we need to **boot something quickly** without hassle... and the manual way of **flashing** one usb with one **operating system** simply does not cut it.

But these days are numbered!

Enjoy your new **Ventoy boot usb stick** and...

**Stay Open!**