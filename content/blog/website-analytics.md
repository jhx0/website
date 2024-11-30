+++
title = "Website Analytics"
date = "2024-11-30T16:44:01+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["nginx", "linux", "goaccess", "analytics", "website", "debian", "ubuntu", "webserver", "website"]
+++

# Foray into analytics
So far, all the **websites** I hosted never had any sort of **analytics** at all - **Never**.   
I think it is high time to change just that.   
This small little guide will leverage **goaccess** to generate static reports that can easily be served via any **webserver** (In this guide, we will use **Nginx**).

# Prepare the directories
First things first - creating the necessary directories.   
This is easily done as follows.

```shell
$ sudo mkdir /var/www/html/metrics
$ sudo chown www-data:www-data /var/www/html/metrics/
```

# One script to rule them all
I've decided to write a small little **script** that will be called from **Cron** so that I do not have to bother running the command by hand.

```shell
$ sudo vim /usr/local/bin/webreport
```

```shell
#!/bin/bash

zcat -f /var/log/nginx/access.log* | \
sudo -u service docker run --rm -v /srv/geoip/db.mmdb:/db.mmdb:ro -i -e LANG=$LANG allinurl/goaccess -a -o html --geoip-database=/db.mmdb --log-format COMBINED - \
> /var/www/html/metrics/report.html

chown www-data:www-data /var/www/html/metrics/report.html
```

Now, make the script is **executable**:

```shell
$ sudo chmod +x /usr/local/bin/webreport
```

That's it for the **script** part.

**Short explanation**:
The script starts a **goaccess** **container** and generates the report. It also leverages **GeoIP** lookups which are explained below (The needed setup for that). For now, you can save the script as is.

# GeoIP shenanigans
We need to grab a suitable **GeoIP database** to make country lookups possible. Luckily this is very easy given the fact that the **Docker** **goaccess** image has **GeoIP** support bultin (Well, the **binary** has support baked in).

```shell
$ wget 'https://github.com/P3TERX/GeoLite.mmdb/raw/download/GeoLite2-Country.mmdb'
```

Next make a dedicated directory and move the **database** over

```shell
$ sudo mkdir /srv/geoip
$ sudo mv GeoLite2-Country.mmdb /srv/geoip/db.mmdb
```

**Disclaimer**: Where you put the **database** does not matter. However keep in mind that the script expects the db to be found under */srv/geoip* - **make the changes needed to suit it to your needs**!

# Cron to the rescue
After all of the above tasks are done we need to configure a **cronjob** to generate the report without interaction periodically.

First, edit */etc/crontab*

```shell
$ sudo vim /etc/crontab
```

Insert the following line:

```shell
30 * * * * root /usr/local/bin/webreport
```

**Explanation**: This will tell cron to run our **script** *webreport* **every 30 minutes**   
(You can of course change this if needed - depending on what you want/need)

**Save your changes** and continue to the **Nginx** setup.

# Webserver time
You could just add the **directory** to your **webserver** of choice and be done with it.     
But if you are anything like me, the fact of having those **statistics** out in the open is not really thrilling.   
A **simple solution** is to use **HTTP authentication** since it is very easy to set up and imho enough to keep prying eyes away.   
So, **let's just do exactly that**!

First, install the **apache2-utils**.
```shell
$ sudo apt install apache2-utils -y
```

Next, **generate** the desired user and set a password.
```shell
$ sudo htpasswd -c /etc/nginx/.htpasswd $USERNAME
```

**Note**: Replace *$USERNAME* with your desired user name.

Enter the **password** twice and we are done with that step.

Now we configure **Nginx** to acutally use **HTTP authentication** on the subdiretory we created in the beginning.

Open the **configuration file of your website**.   
(This varies wildly depending on how you named it and how you did configure nginx!)

Add the following lines to your **vhost** **configuration** for your site

```shell
location /metrics {
    auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;

    index report.html;
}
```

**Explanation**: This tells **Nginx** that you serve the subfolder */metrics* with **HTTP** authentication.    
Also, the **index** file was set to the created report from **goaccess** - which is named **report.html** in the **script**.

Reload **Nginx** to apply the changes.
```shell
$ sudo systemctl reload nginx
```

# Analytics all the way!
The only thing left to do is to **navigate to the report you just created**!   
Use your **browser** of choice here - I'll use **Firefox** here.

```shell
$ firefox https://website.com/metrics
```

**Note**: Before anything will be displayed the **cronjob** has to be run before to actually **generate the report**! - **Keep that in mind**!

**Done**. You now have the power to gain more insight into your **website**.

# Final words
**goaccess** really makes gathering analytics easy and painless. Given the fact that you can just run a quick **Docker** oneliner and be done is for sure comfy.

You might want to have a look at the options **goaccess** provides to further improve your setup. Or you might want to alter the script...

**No matter what**: You can do so according to your own needs.

Last but not least...

**Stay Open**!