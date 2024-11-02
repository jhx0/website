+++
title = "Hosting the Arch Wiki locally"
date = "2024-11-02T16:52:02+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["archlinux", "wiki", "debian", "ubuntu", "hosting", "lighttpd", "docs", "webserver" ]
+++

# Arch Wiki
Everyone knows and loves it. **The one and only Arch Wiki**.   
There is so much information encapsulated in the **Wiki** that having a browsable **local copy** can be quite beneficial.

# Requirements
The only real requirement for hosting the **Arch Wiki** locally is a **webserver**.   
There are plenty of choices out there, for example **NGinx** or **Apache** - just to name two.   
In this guide we will leverage **Lighttpd** since it is easy to use and also very light on resources.

# Lighttpd
First step is installing **Lighttpd**.   
To install, one only needs to leverage there **package manager**. In this example we will target **pacman** (**Arch Linux**) and **apt** (**Debian** / **Ubuntu** based **distros**).

**Arch Linux:**
```shell
$ sudo pacman -S lighttpd
```
**Debian / Ubuntu:**
```shell
$ sudo apt install lighttpd
```
**Step one is done**.   
In the next step we will get a copy of the current **Arch Wiki** for **offline** serving / reading.

# Getting a copy of the wiki
**Disclaimer:** **Arch Linux** users can simply **install the package**. There is **no need** to download the package manually! (**Keep that in mind**)

Navigate with your **browser** of choice to the following **URL** (**Firefox** in the example):
```shell
$ firefox https://archlinux.org/packages/extra/any/arch-wiki-docs/
```
![Arch Wiki Package](/images/arch-wiki-package.png)   
On the **right side** you will find the "**Download From Mirror**" **Link** which allows you to save a copy of the package locally.   
You can also **directly get a package** by leveraging the excellent **curl** utility as follows:
```shell
$ https://de.arch.mirror.kescher.at/extra/os/x86_64/arch-wiki-docs-20241101-1-any.pkg.tar.zst -o arch-wiki-docs.pkg.tar.zst
```
**Hint:** This is a **direct link** to the **current package**. Make sure to grab the **lastest package** always!

## Extracting the package
We now have the current package downloaded and **Lighttpd** is installed.   
The next step is to **extract the archive** so that we can tell the **webserver** which **root directory** he will serve.

First, I'll create a **directory** which will hold the files.
```shell
$ sudo mkdir /srv/wiki
```

Next, **extract** the **archive**.
```shell
$ sudo tar xf arch-wiki-docs.pkg.tar.zst -C /srv/wiki
```

**List all the files** and make sure the extraction did work
```shell
$ ls /srv/wiki/usr/share/doc/arch-wiki/html/
```

**Perfect**, we are now ready to **configure Lighttpd**.

## Webserver shenanigans
**For Debian / Ubuntu users:**   
Now that we got a **local copy of the Arch Wiki** it is time to finalize our little adventure by altering the **Lighttpd configuration** to our liking.   
Edit "**/etc/lighttpd/lighttpd.conf**" and add the following line to it:
```shell
...
server.document-root = "/srv/wiki/usr/share/doc/arch-wiki/html/"
```
Make sure the **path** to the HTML files is actually **correct**!

**For Arch Linux users:**   
After **installing** the Arch Wiki package the contents of said package are extracted to "**/usr/share/doc/arch-wiki/html**".   
```shell
$ sudo pacman -Syy arch-wiki-docs
```
The only thing left is pointing the **webserver** to the right **location**.   
Edit "**/etc/lighttpd/lighttpd.conf**" and alter **server.document-root** to point to the right **directoy**.
```shell
server.document-root = "/usr/share/doc/arch-wiki/html/"
```

It is now time to reload the **webserver** and test if all has gone well.
```shell
$ sudo systemctl restart lighttpd
```

## Entering the Matrix... I mean the Arch Wiki
Use your **browser of choice** and enter the following **URL** - Assuming you did not change anything except the **document-root**.
```shell
$ firefox http://localhost
```
![Lighttpd Arch Wiki](/images/lighttpd-arch-wiki.png) 

## Final words
**Hosting a local copy of the Arch Wiki** can be quite helpful in many ways.   
The turning point for me was being able to read the **wiki** if there are problems with my **Internet access** at home or when I need to **lookup something on the go**.   
There are quite some **possibilities** given, for example, **searching the wiki via commandline tools** or simply **browsing the wiki for interesting new things**.

Hope you enjoy your **local wiki copy**!

Last but not least: **Stay Open!**