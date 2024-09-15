+++
title = "OPNSense Automated Config Backup"
date = "2024-09-15T13:59:53+02:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["opnsense", "firewall", "backup", "script", "freebsd", "scp", "ssh"]
+++

# OPNSense and backups
I lately searched for a way to do automated config backups which will be stored directly on my **NAS**.   
The **Business Edition** of **OPNSense** allows to do centralized backups, given the **firewall** in question is a managed via **OPNCentral**.   
In my **homelab**, this is not the case.   
To solve this very issue I've set down and thought what I can do. It turns out the process is quite easy and can be managed in **under 10 minutes** - if all goes well (*The devil is in the details as usual*).

# Preparing the NAS
The first thing to do is creating a **user** and **group** which we leverage when using **scp** on the **firewall** in question.
For this I leverage 'pw' - you could also use 'adduser' on FreeBSD, depending on what you so desire.

```bash
$ sudo pw groupadd -g 2000 backup
$ sudo pw user add -n backup -c 'Backup User' -d /data/Backup  -m -s /bin/sh -g 2000
```

This gives us the user '*backup*' which has a **group** of the same name.

The next thing to do is generating the needed **SSH key pair** for the user to be able to login without giving a password - since the script later on will run from **Cron** (*No interactivity*).

# SSH
On the **OPNSense** in question, we run the following command to **generate a key pair**.

```shell
$ sudo ssh-keygen -t rsa -b 4096 -C "backup@seven.lan" -f /usr/home/id_rsa
```
This will create a **public**/**private key pair** and save both keys under */usr/home*.
**Note:** You could of course select another destionation - that is up to what you need.

The only thing left to do here is to add the **public key** to the user backups **authorized_keys** file on the destionation (**NAS**).

```shell
$ sudo cat /usr/home/id_rsa.pub
```

Then on the **destination**, add the key to the **authorized_keys** file in the users home directory (**/data/Backup** in this case - *the home directory of the backup user*).
Make sure the .ssh directory is created. If that is not the case, simply create it with the **correct permissions**.

```shell
$ mkdir /data/Backup/.ssh && touch /data/Backup/.ssh/authorized_keys
```

With that all set the fun continues with the actual meat and potatoes of the whole process.

# Setting up the backup script
First create the file **backup.sh** (**script**).

```shell
$ sudo vi /usr/home/backup.sh
```

and paste in the following **content**.

```shell
#!/bin/sh

REMOTE_SERVER='10.0.5.100'
REMOTE_USER='backup'
REMOTE_BAK_DIR='firewall/'
REMOTE_SSH_PORT='10700'
FW_BAK_PATH='/conf/backup/'
TARGET_NAME="`hostname`-`date +%d-%m-%Y-%H-%M-%S`-backup.xml"
SCP_OPTIONS="-P $REMOTE_SSH_PORT -o StrictHostKeyChecking=no"
CONF_FILE=${FW_BAK_PATH}`ls -tr $FW_BAK_PATH | tail -n1`
SSH_KEY='/usr/home/id_rsa'

scp -i $SSH_KEY $SCP_OPTIONS $CONF_FILE ${REMOTE_USER}@${REMOTE_SERVER}:${REMOTE_BAK_DIR}${TARGET_NAME}
```
Adjust the script to your own needs. Especially the **user name** and **SSH port**.
Also, make sure to set the **executable bit** on the script you have just created!

```shell
$ sudo chmod +x /usr/home/backup.sh
```

On to the **next chapter**.

# Defining a custom action
To be able to shedule a **cronjob** with our newly created **script**, we have to define a new action.   
Create the following **file**.

```shell
vi /usr/local/opnsense/service/conf/actions.d/actions_seven-backup.conf
```

Paste in the following **content**.

```shell
[seven-backup]
description:Seven Remote Backup (Config)
command:/usr/home/backup.sh
type:script
message:Taking config backup
```
**Again**, change this to your own needs!
To make **OPNSense** recognize the **action** we've just defined, we need to restart the **configd** service.

```shell
$ service configd restart
```

# Cron fun
Now that we have the **backup script** and **action** defined we can continue to create a new **cronjob**.

Log in to the **OPNSense WebGUI** and navigate as follows.

```
System -> Settings -> Cron
```

![OPNSense Cronjobs](/images/opnsense-cron.png)

Via the '**+**' Button you need to create a **new cronjob** - see the attached **screenshot below**.

![OPNSense Add Cronjob](/images/opnsense-add-cronjob.png)

In this very example the **backup.sh script** would run every day at **4 AM** to **upload** the current config to the **NAS**.
Be sure to **adjust** the **time** as needed!

# Final words
All in all, the process is **fairly simple** and not too involved. It is a easy way to **automate taking backups** with less effort. In the end, it all depends on your own needs.   
Having for example **Business editions** rolled out would make it possible to have a central **OPNSense** (**OPENCentral**) that gathers backups regularly from joined OPNSense firewalls out in the wild - **YMMV**.

That's it. **Have fun everyone!**