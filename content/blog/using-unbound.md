+++
title = "Using Unbound"
date = "2025-11-24T13:53:54+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["linux", "debian", "dns", "unbound", "dot", "tls",]
+++

We all know that **DNS** is the lifeblood of the modern **Internet**.    
But, we also all know that resolving **DNS** queries can take some time and by default queries are sent unencrypted to a outside **DNS** server - if one does not selfhost **DNS** of course.   
In this little post I set up **Unbound** with **DoT**. Also, **Unbound** will cache results which makes queries faster in the future (Since these don't need to be resolved again)   

**Disclaimer**: Of course, entries (**Cache**) cannot exist forever, that would be kinda bad. Keep that in mind!

# Installation
Installing **Unbound** is easy and should be present in any major **Linux** distribution or **BSD** derivative.   
In this case I will use **Debian** (Which will also cover **Ubuntu** and derivatives).   
Let's fire up a terminal emulator and use **apt** to install **Unbound**

```bash
$ sudo apt install -y unbound
```

That's it! We can go straight over to configuring **Unbound**.

# Configuration fun
In **Debian** there is a central **Unbound** config which sources other config snippets from the directory `/etc/unbound/unbound.conf.d/`.   
All we have to do is create a new file in said directory structure (**Include directory**) and paste our configuration directives.   
I will use **vim** as a editor - pick what you like best!

```bash
$ sudo vim /etc/unbound/unbound.conf.d/local.conf
```

Next, paste in our config snippet.

```bash
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

  tls-cert-bundle: /etc/ssl/certs/ca-certificates.crt

  forward-zone:
    name: "seven.lan."
    forward-addr: 10.0.5.50

  forward-zone:
    name: "."
    forward-ssl-upstream: yes
    forward-addr: 1.1.1.1@853#cloudflare-dns.com
```
So far, so good... but what does this config **do**?   
Let's go over some of the **entries** here.

`username` = The user **Unbound** is running as   
`directory` = **Unbound** directory   
`do-ip6` = Use **IPv6**? yes or no - Make sure to alter this if you use **IPv6**!   
`interface`, `port` = The interface and port to bind to   
`prefetch` = Keep the cache up to date   

`verbosity`, `log-queries`, `log-replies` = Log queries. **Beware**: This will generate a lot of log traffic! (Should only be used for testing imho)   
`cache-max-ttl`, `cache-min-ttl` = Min/Max **TTL** (**Time To Live**)

`hide-identity`, `hide-version` = Don't give away the version number/identity

`domain-insecure` = I have a local domain, seven.lan, which needs to be set as "insecure" so **Unbound** forwards queries to the given **DNS Server**. (There is a **forward-zone** delcared for my own DNS server below)   
**Note**: Disable this if you do not leverage another **DNS Server** which should handle queries to a local zone!

`tls-cert-bundle` = Certificate bundle - Needed for **DoT**

The bottom two **forward-zone** entries handle first my local DNS server and second a catch all **"."** where all other queries go to.
```bash
    ...
    forward-ssl-upstream: yes
    forward-addr: 1.1.1.1@853#cloudflare-dns.com
```
`forward-ssl-upstream` = Forward queries (TLS)   
`forward-addr`= The DNS server to use. In this case i picked **Cloudflare** on the **port 853** (Which is the **DoT** port used)

**Alright**, the configuration part is done.   
Let's head over to the **next step**!

# Handling Unbound
First, we need to make sure to **load** or just created **config file**.
```bash
$ sudo systemctl restart unbound
```

Let's see if **Unbound** is indeed running.
```bash
$ systemctl status unbound
```
```bash
● unbound.service - Unbound DNS server
     Loaded: loaded (/usr/lib/systemd/system/unbound.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-11-24 13:47:54 CET; 32min ago
 Invocation: f596a608512c4ddd9632a844cf4deea4
       Docs: man:unbound(8)
    Process: 1497 ExecStartPre=/usr/libexec/unbound-helper chroot_setup (code=exited, status=0/SUCCESS)
    Process: 1501 ExecStartPre=/usr/libexec/unbound-helper root_trust_anchor_update (code=exited, status=0/SUCCES>
   Main PID: 1504 (unbound)
      Tasks: 1 (limit: 2265)
     Memory: 11.2M (peak: 13M)
        CPU: 73ms
     CGroup: /system.slice/unbound.service
             └─1504 /usr/sbin/unbound -d -p
```
Everything **looks good** so far!   
**But**: We currently do not use **Unbound** for any DNS queries - all we did so far is install/configure **Unbound** itself. We did however never tell our system to use it for queries.   
So, let's fix that!

# resolv.conf
All we have to do is tell the system about our **DNS resolver**.   
To do that we have to edit `/etc/resolv.conf`.

**Note**: Your networking configuration will most likely by different than mine! You might use **NetworkManager**, **Netplan**, **Ifupdown**, etc. Make sure to change your preferred DNS server in the correct places according to your needs!

Time to **edit** the file
```bash
$ sudo vim /etc/resolv.conf
```
...and make sure it looks something like this
```bash
...
nameserver 127.0.0.1
```
Since `interface` was set to **"127.0.0.1"** Unbound will listen on that address (On **port 53** - you can check this with `ss -l4`).

All set, let's do some **DNS queries**!

# Talking to DNS
Simple first test: Let's ping the **OpenBSD** projects website which will trigger a **DNS query** (To resolve the domain).

```bash
$ ping openbsd.org
```
```bash
x@vdeb:~$ ping openbsd.org
PING openbsd.org (199.185.178.80) 56(84) bytes of data.
64 bytes from 199.185.178.80: icmp_seq=1 ttl=243 time=168 ms
64 bytes from 199.185.178.80: icmp_seq=2 ttl=243 time=166 ms
```
Lookin good so far.   
**Another** way of checking is leveraging **dig** for example.
```bash
$ dig @127.0.0.1 -p 53 kernel.org
```bash
; <<>> DiG 9.20.15-1~deb13u1-Debian <<>> @127.0.0.1 -p 53 kernel.org
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 14126
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;kernel.org.                    IN      A

;; ANSWER SECTION:
kernel.org.             1200    IN      A       139.178.84.217

;; Query time: 52 msec
;; SERVER: 127.0.0.1#53(127.0.0.1) (UDP)
;; WHEN: Mon Nov 24 14:52:02 CET 2025
;; MSG SIZE  rcvd: 55
```
**Note**: The DNS resolver is explicitly given, 127.0.0.1, and also the port is specified as well (53 - **DNS**).

But I want to make sure that my queries actually use **DoT**.   
Let's fire up **tcpdump** and see what we get.
```bash
$ sudo tcpdump port 853
13:50:20.641067 IP vdeb.void.lan.51404 > one.one.one.one.domain-s: Flags [S], seq 1097630260, win 64240, options [mss 1460,sackOK,TS val 605651785 ecr 0,nop,wscale 7], length 0
13:50:20.655910 IP one.one.one.one.domain-s > vdeb.void.lan.51404: Flags [S.], seq 819511098, ack 1097630261, win 65535, options [mss 1452,sackOK,TS val 2095896701 ecr 605651785,nop,wscale 13], length 0
13:50:20.655954 IP vdeb.void.lan.51404 > one.one.one.one.domain-s: Flags [.], ack 1, win 502, options [nop,nop,TS val 605651799 ecr 2095896701], length 0
13:50:20.656425 IP vdeb.void.lan.51404 > one.one.one.one.domain-s: Flags [P.], seq 1:1561, ack 1, win 502, options [nop,nop,TS val 605651800 ecr 2095896701], length 1560
```
I did the same **ping** example we did above and saw, while having **tcpdump** open, that there are queries to **port 853** which is the port used in **DoT**.
All well so far!

We can also check the **Unbound** **journal**/**log** to see what query we have made.
```bash
$ sudo journalctl -f -u unbound
```
```bash
Nov 24 13:48:04 vdeb unbound[1504]: [1504:0] info: 127.0.0.1 openbsd.org. A IN
Nov 24 13:48:04 vdeb unbound[1504]: [1504:0] info: 127.0.0.1 openbsd.org. AAAA IN
Nov 24 13:48:05 vdeb unbound[1504]: [1504:0] info: generate keytag query _ta-4f66-9728. NULL IN
Nov 24 13:48:05 vdeb unbound[1504]: [1504:0] info: 127.0.0.1 openbsd.org. AAAA IN NOERROR 0.296568 0 57
Nov 24 13:48:05 vdeb unbound[1504]: [1504:0] info: 127.0.0.1 openbsd.org. A IN NOERROR 0.296568 0 45
Nov 24 13:48:05 vdeb unbound[1504]: [1504:0] info: 127.0.0.1 80.178.185.199.in-addr.arpa. PTR IN
Nov 24 13:48:05 vdeb unbound[1504]: [1504:0] info: 127.0.0.1 80.178.185.199.in-addr.arpa. PTR IN NXDOMAIN 0.179146 0 102
```
We can see from the output that indeed there have been queries made by **Unbound** to resolve the domain in question (Records: **A** [IPv4] and **AAAA** [IPv6])).

**Alright**, all is working as intended - **we are done at this point**!

# Final words
**Unbound** of course has many more options which can be leveraged.   
In the end, we got **Unbound** doing or **DNS queries** and made the whole process more secure by leveraging **DNS over TLS**.   
Make sure to check out the `unbound.conf` **manual page** for more options.   
https://unbound.docs.nlnetlabs.nl/en/latest/manpages/unbound.conf.html

This was it for our little excursion of **Unbound** - **A fantastic tool** to have!

**Enjoy**!

...and as always:

**Stay Open!**