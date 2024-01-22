+++
title = "Creating a Swapfile"
date = "2023-11-12T13:52:38+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["linux","swap",]
+++

Creating a **swapfile** can be easily done with **dd**.

```shell
$ sudo dd if=/dev/zero of=/swapfile bs=1M count=4k
```
This will create a **4GB** file directly under the **/** (**root**) directory.   
**Hint**: **count** specifies how many **1M** (**bs**) blocks will be allocated.

After the creation make sure to alter the **permissions** of the created file.

```shell
$ sudo chmod 600 /swapfile
```

Now follows the actual formatting of the given file.

```shell
$ sudo mkswap -U clear /swapfile
```

Activate the **swapfile** after formatting.

```shell
$ sudo swapon /swapfile
```

The only thing left to do is to mount the **swapfile** on every **boot**.
To do this we will add the following to the systems **fstab** file (**/etc/fstab**).

```shell
$ sudo vim /etc/fstab
```

Append the following to your **fstab** file:

`/swapfile none swap defaults 0 0`

Done, the next time the system boots the **swapfile** will automatically be used.

**Further reading**: https://wiki.archlinux.org/title/swap