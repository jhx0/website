+++
title = "Debian PXE"
date = "2023-12-27T11:35:14+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["debian","pxe", "tftp"]
+++

# Why?
Having to install  [**Debian**](https://debian.org) from time to time really made me think to doing the install over the network, rather than always making USB sticks or mounting a **ISO** file in a **VM**.   
The steps to set up [**PXE**](https://en.wikipedia.org/wiki/Preboot_Execution_Environment) on **Debian** are quite easy - all in all maybe a 10 minute process.   

...**So**, let's get to it!

# Requirements
You need a working **Debian** install - That's it.   
Everything else can be installed from the official repositories.

# Setting up TFTP
The first thing on the agenda ist installing a [**TFTP**](https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol) service/daemon.   
I will use [**tftpd-hpa**](https://packages.debian.org/bookworm/tftpd-hpa) - you can of course use something else like dnsmasq for example (Which has TFTP builtin).

```shell
$ sudo apt install tftpd-hpa
```

# Configuring tftpd-hpa
Here is a example of how to configure the service. (Be aware to change the given **IP address** to your needs!)

```shell
$ sudo vim /etc/default/tftpd-hpa
```

`TFTP_USERNAME="tftp"`   
`TFTP_DIRECTORY="/srv/pxe"`   
`TFTP_ADDRESS="10.0.5.90:69"`   
`TFTP_OPTIONS="--secure -4 -v"`


### Explanation: 
**TFTP_OPTIONS:** This example only uses IPv4 (-4) and -v (verbose).      
**TFTP_ADRESS:** Specifies the IP address to listen on   
**TFTP_DIRECTORY:** This is the path were we later store the necessary files 

# Creating the PXE directory
Depending on your configuraiton you might need to create the **directory**.   
For me:

```shell
$ sudo mkdir /srv/pxe
```

# Restart tftpd-hpa
Restart the **service** after all the configuration is done.

```shell
$ sudo systemctl restart tftpd-hpa.service
```

# Preparing the needed files
Change to the created **directory**, **download** the needed files and **extract** them.

```shell
$ cd /srv/pxe
$ sudo wget https://deb.debian.org/debian/dists/bookworm/main/installer-amd64/current/images/netboot/netboot.tar.gz
$ sudo tar xzvf netboot.tar.gz
```

# Done
You can now boot a **Debian installer** over the **network**.   
Make sure to set your desired system to boot over network.   
(Most likely there is a option in the [**BIOS**](https://en.wikipedia.org/wiki/BIOS) that you can switch on)

# UEFI
Making a [**UEFI**](https://en.wikipedia.org/wiki/UEFI) system **bootable** over the network is a simple as linking the correct **bootloader** (**GRUB**) in the created directory.

```shell
$ cd /srv/pxe
$ ln -s debian-installer/amd64/grubx64.efi .
$ ln -s debian-installer/amd64/grub .
```

# Configure your DHCP service
Your [**DHCP**](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) server of choice must be configured to point at the **TFTP** server/service. (This will make it possible to **download** the required files!)   

I personally use [**OPNSense**](https://opnsense.org/) - as seen on the following screenshot.

![OPNSense](/images/opnsense-example-tftp.png)

Alter the **TFTP server** options (**TFTP hostname** and **Bootfile**) and you will be ready to go.

**Beware**: Keep in mind specifying the right boot file!

**BIOS** - `pxelinux.0`   
**UEFI** - `grubx64.efi`

# Closing words
Setting up a Debian host to do **PXE** booting is quite simple and saves a lot of hassle.   
No more making USB sticks let alone CD/DVD's!