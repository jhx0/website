+++
title = "Serving ISO's over HTTP"
date = "2025-11-27T14:45:25+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["freebsd", "lighttpd", "iso", "share", "download", "bsd",]
+++

We are sitting on our computer somewhere and are in the need of an **ISO**.   
Maybe we want to create a virtual machine or have some other need like flashing said **ISO** to a usb stick.   
Sometimes it is too much hassle to mount our **NAS** share or we are on a totally different device which is not ours.   
I've had quite some situations in which I was working on a **PC**/**Laptop** that a friend gave me on which I did not want to mount my network share.   

So, **what can we do**?

The answer is **simple**. We serve our **ISO's** via **HTTP**!

# Webserver
**Which one** do we chose?   
This is personal preference. But for a simple task like this I personally like **Lighttpd** which is easy to set up and works well.

First, we need to install **lighttpd**.
```bash
$ sudo pkg install -y lighttpd
```

After the installation is finished we enable the service.
```bash
$ sudo sysrc lighttpd_enable="YES"
```

Before we start **Lighttpd** we should first configure it.   

# Configuration fun
First things first. We want to access our **ISO's** over a custom **DNS** name - A so called **A Record** in **DNS** parlence.   
To do that we will first create a directory that holds our configuration file.
```bash
$ $ sudo mkdir /usr/local/etc/lighttpd/vhost
```
After that is done we need to edit the **Lighttpd** configuration file.

```bash
$ sudo vim /usr/local/etc/lighttpd/lighttpd.annotated.conf
```

Add the following lines at the end of the file.
```bash
...
# Include custom vhost config directory
include conf_dir + "/vhost/*.conf"
```

This will make it possible to add config snippets to **Lighttpd** without editing the main configuration file. Technically, the line we've added specifies that **Lighttpd** should read in configuration files from the `vhost` directory (The files need to end in `*.conf` - **Keep that in mind**!)

Let's move on to the actual config of our site.
Create a new file under the `vhost` directory and paste in the following contents.
```bash
$ sudo vim /usr/local/etc/lighttpd/vhost/iso.conf
```
```bash
$HTTP["host"] =~ "iso\.void\.lan" {
        server.document-root = "/data/iso"
        dir-listing.activate = "enable"
}
```
Let me explain a little.   
This snippet specifies that when a request enters our webserver with the hostname `iso.seven.lan` it should set the **Document Root** to `/data/iso` and it should enable **Directory Listings**.   
**Note:** `/data/iso` is my local **ISO** directory where all my files are stored. You need to change this to the location where your **ISO's** are saved!

To check if our configuration was successful we can run **Lighttpd** with certain flags - this will dump all configuration parameters to **stdout** for us to inspect.
```bash
$ sudo lighttpd -p -f lighttpd.annotated.conf | less
```
You should see the above config snippet at the end of the output.

**Note on DNS**: You can select whatever hostname fits your bill - there are no limits. However, you need to make sure that the given **A Record** or **CNAME** entry is resolveable! Configruing a **DNS** server is out of the scope of this text - **Bind** is a popular **DNS** server for example.

**Quick and dirty**: If you just want to test out that it works you can of course make a simple entry in your `/etc/hosts` file. Simple and quick!

# Give me permision
The final step before starting **Lighttpd** is to check the permissions of the files we want to serve - and the directories for that matter!

In my case I need to **chown** the directories in question to make it able for the user `www` (Which **Lighttpd** runs as) to read files inside these directories.
```bash
$ sudo chown :www /data
$ sudo chown :www /data/iso
```
**Note**: Make sure your permissions are correctly set! To read a file you will need `r` and `x` for traversing directories!

# Let the show begin!
It is now time to start **Lighttpd**.
```bash
$ sudo service lighttpd start
```

The moment of truth is here, finally.
Let's fire up or browser of choice.
```bash
$ firefox http://iso.void.lan
```

![ISO site](/images/lighttpd-iso-dir.png)

**Et voila**, the site works and we can now access our **ISO's** over **HTTP** in case we need something quickly!

# Last words
Sure, **Lighttpd** can do much more - that is a fact. We've barely scratched the surface here.   
**But**: We don't need more than that. In the end, the task at hand is simple. Simple tasks need simple solutions after all!

Have a look at the **documentation** and **MAN** pages for further configuration.

Enjoy!

...and as always:

**Stay Open!**