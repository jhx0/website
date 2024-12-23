+++
title = "Local Unbound Setup"
date = "2024-12-23T18:26:09+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["dns", "cache", "unbound", "debian", "ubuntu", "domain"]
+++

I thought it would make sense to run **Unbound** locally to cache my **DNS queries** - which can **speedup DNS resolution** quite a bit.   
So, **let's get started!**

# Installing Unbound
**Unbound** is available on all major **Linux** distributions and on the various **BSD's**. I use **Debian** in this case and install **Unbound** via **apt**.

```shell
$ sudo apt install unbound -y
```

Use the **package manager** of your choice to install **Unbound**. (**pkg** on **FreeBSD** for example or **pacman** on **Arch Linux** - Whatever you might have on hand)

**That is literally it** - we can now shift over to **configuring Unbound**.

# Configuration fun
Under **Debian** (or **Debian** based distributions) you will find the needed **configuration files** under:
```shell
/etc/unbound/
```
You could move on the edit */etc/unbound/unbound.conf* directly, but I will create a **new configuration file** under */etc/unbound/unbound.conf.d* since files in this directory are **included automatically** (This also keeps the main configuration file clean).

```shell
/etc/unbound/unbound.conf.d/config.conf
```
**NOTE:** You can name the file any way you want. Just make sure it ends in ***.conf**.

We can now paste in the following **directives**:
```shell
server:
  username: "unbound"
  directory: "/etc/unbound"
  do-ip6: no
  interface: 127.0.0.1
  port: 53
  prefetch: yes

  verbosity: 1
  log-queries: yes
  log-replies: yes

  cache-max-ttl: 14400
  cache-min-ttl: 1200

  hide-identity: yes
  hide-version: yes

  domain-insecure: "seven.lan"

  forward-zone:
    name: "seven.lan."
    forward-addr: 10.0.5.50

  forward-zone:
    name: "."
    forward-addr: 10.0.5.50
```
**EXPLANATION:**
- **username**: The user which runs **Unbound** (*"unbound"* under **Debian**)
- **do-ip6**: I don't currently use **IPv6**, so this is set to "no"
- **interface**: I bind **Unbound** to localhost only - change this to your needs.
- **port**: 53, nothing to change here
- **prefetch**: Enable prefetching
- **verbosity**: Set to "1" to get more detailed log outputs (**Journal**)
- **cache-***: Setting the cache **lifetime in seconds**
- **hide-***: Hide version and identity - Useful when **hardening**
- **domain-insecure**: This is for my local domain which needs to be excluded for querying
- **forward-zone**: Define all the zones and the forwarder

In my setup, I **forward queries** to a local **Bind** instance which runs as a **authorative DNS** server. This might be something you don't need!   
Make sure to leave out *"domain-insecure"* and the above listed *"forward-zone"* for my domain. You only need *"."* (All) as a **forward zone** with a **public DNS** (Which means all queries are **forwarded** to that **DNS** server)

Save the configuration file and **restart Unbound**.
```shell
$ sudo systemctl restart unbound"
```

# Testing
Now that we have set up **Unbound** we need to make sure that it works actually.    
Let's first display the **journal** in one terminal tab and then do a query.
```shell
$ sudo journalctl -f -u unbound
```
**And now**, let's test if all works
```shell
$ dig openbsd.org @127.0.0.1
```
We can see that from the output (**Journal**) the **query was successful**!
```shell
...
Dec 23 18:30:11 chaos unbound[1189013]: [1189013:0] info: 127.0.0.1 openbsd.org. A IN
```

**Perfect, it works!**

# Conclusion
Setting up **Unbound** is quite simple for the most part.   
This is by the way a very easy setup - Nothing complex or complicated. It sure makes sense to **cache DNS queries** to **speedup** the lookup process at a later time.

**Have fun** playing around with **Unbound**!

**Stay Open!**