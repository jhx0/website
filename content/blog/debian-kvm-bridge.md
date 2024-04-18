+++
title = "KVM and Docker IPTables fun"
date = "2024-04-18T16:21:41+02:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["debian","kvm","bridge","qemu","vm","virtualization",]
+++

# Debian: Docker and KVM
Virtualization under **Debian** is a breeze. Install the needed packages and off you go...

**Except for one fact:**   
If you want to use a **bridge** on your **vm's** and you also happen to run **Docker** you might have run into a wall...   
That being that your **vm's** don't have internet connectivity/networking available.

# Docker
**Docker** creates **IPTables** rules which by default don't allow forwarding for anything except **Docker** created containers.   
So, let us tackle the problem and make your **bridge** device usable again!

# IPTables fix
After you have **Docker** installed and **KVM/QEMU/Libvirt** installed you need to just run one command:

```shell
$ sudo iptables -A FORWARD -i br0 -o br0 -j ACCEPT
```

**What** does it **do**?   
Well, simple...
It allows **packet forwarding** to your **bridge** device, here **br0**.

# So, all is well now?
**No**, not quite.   
When you **reboot** your system this very rule we've just set is **not persistent** - which means it will be gone next time.      
A easy solution, that I use myself, is installing the **iptables-persistent** **package**. The **package** asks after installation to save the current **ruleset** to a file so it can restore the setting later, for example, after a **reboot**.   

**Installing** is a simple as:

```shell
$ sudo apt install iptables-persistent -y
```

Next, follow the instructions on screen (**debconf**) to save the current **ruleset**.   
**That's it** - The issue is now taken care of.

# Is this the only solution?
**Not at all**.   
You could also setup a script or use a different type of networking if so desired. Also, you could setup **bridging** via **NetworkManager** if you so desire.   
This is merely the way I do it - **YMMV**.