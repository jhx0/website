+++
title = "FreeBSD Nas"
date = "2023-11-19T13:27:16+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["freebsd","nas","nfs"]
+++

# Motivation
I personally use vanilla [FreeBSD](https://freebsd.org) on a **low power PC** to handle all my data needs.
This setup has served me well over the time, which is why I want to dedicate a few lines of text to it.

# Assumption
I assume that the reader uses **sudo** in this article. You can of course become **root** by using **su** or you might leverage **doas**. This is up to you. **sudo** works well for me personally, **YMMV**.

# FreeBSD
The installation itself is very easy to do and takes just a couple of minutes.
There is nothing special here, it is your standard **FreeBSD** install with the settings/tweaks you want to do.
I, in general, leverage [ZFS](https://docs.freebsd.org/en/books/handbook/zfs/) for the **root** (**/**) filesystem - but this is not a requirement. (Chose what works for you)

# ZFS setup
Currently I have two **8TB** (Two different manufacturers: **Seagate** and **WD**) drives in there for my storage needs. The most important part here is creating a **ZFS Mirror**.

First, I determine what **devices** I have to work with.

```shell
$ sudo camcontrol devlist
``` 

Output:

><ST8000DM004-2U9188 0001>          at scbus0 target 0 lun 0 (pass0,ada0)  
<Samsung SSD 750 EVO 250GB MAT01B6Q>  at scbus1 target 0 lun 0 (pass1,ada1)  
<HGST HUS728T8TALE6L4 V8GNW9G0>    at scbus2 target 0 lun 0 (pass2,ada2)  
<AHCI SGPIO Enclosure 2.00 0001>   at scbus4 target 0 lun 0 (pass3,ses0)

You can see here that there are two **hard disks** currently in the system (The system disk being a **SSD**).

Next, creating the **ZFS Mirror**:

```shell
$ sudo zpool create nas mirror /dev/ada0 /dev/ada2
```

The array is now mounted under **/nas**. From this point on I can set up all the necessary **software**.  

One thing left to do is setting the **permissions** of the newly created filesystem as follows.

```shell
$ cd /nas
$ sudo chown x:x -R .
```
**Beware:** Make sure you pick the correct **username**/**uid**.

# NFS

Now it is time to setup [NFS](https://docs.freebsd.org/en/books/handbook/network-servers/#network-nfs) and the given **share(s)**.  

I start by editing **/etc/exports**.

```shell
$ sudo vi /etc/exports
```

Following, the content of my **/etc/exports** file.

`
/nas 10.0.7.5
`

Simple stuff in fact.

Last thing to do is configuring **/etc/rc.conf.local**.  

**Hint**: You could use **/etc/rc.conf** directly, but separating your changes from the main **rc.conf** file is a good thing.

Added lines to **/etc/rc.conf.local**:

>rpcbind_enable="YES"  
nfs_server_enable="YES"  
mountd_enable="YES"

I could have started the **NFS** service at this point and be done, but I always **reboot** the system to make sure everything is on point.

```shell
$ sudo reboot
```

After the machine is back up again I will mount the **NFS share** over on my desktop/laptop system.

```shell
(client)$ sudo mount -t nfs 10.0.5.50:/nas /mnt/nas -o rw
```

**Success!** Everything is pretty much ready to go.  

One last final thing to do, or what I do, is edit the **/etc/fstab** file on my system to auto mount the share.

```shell
(client)$ sudo vim /etc/fstab
```

Adding the following line.

`10.0.5.50:/nas /mnt/nas nfs rw,nofail 0 0`

# Final Notes
In general, the possibilities are endless. **FreeBSD** does provide much more than what is described here.  
To be fair, this is more of a **"quick and dirty"** howto to get the ball rolling.

# Further reading
Handling disks under **FreeBSD**: https://docs.freebsd.org/en/books/handbook/disks/