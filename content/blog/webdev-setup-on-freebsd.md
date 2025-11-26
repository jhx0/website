+++
title = "Webdev Setup on FreeBSD"
date = "2025-11-25T14:36:45+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["freebsd", "webdev", "apache", "php", "unix", "webserver", "bsd", "python",]
+++

I wanted a small **webdev setup** where I could test some **Python CGI** code and do some shenanigans with **PHP**.   
Nothing special here and certainly nothing professional - **Not a webdev by trade at all**!    
Let's delve into setting up that **environment**.

# Apache all the way
The **webserver** of choice for me is **Apache** in this setup. Sure, one can use **Nginx**, **Lighttpd** or something else entirely - that is up to the reader.   
First things first: We need to install **Apache** and **PHP**
```bash
$ sudo pkg install -y apache24 php85 mod_php85
```
**Hint**: **FreeBSD** provides more PHP versions if the latest is not wanted. **Keep that in mind**!

And now we enable and start the **service**.
```bash
$ sudo sysrc apache24_enable="YES"
$ sudo service apache24 start
```

We now have some things to take care of before calling it good.    
I want a directory in my **$HOME** folder which **Apache** serves to me and the files therein.   
First create the directory which will hold our files for **Apache** to serve.
```bash
$ cd ~
$ mkdir public_html
```
After that we need to tell **Apache** to serve the given directory.   
Open `/usr/local/etc/apache24/httpd.conf`with the editor of your choice and uncoment the following lines.
```bash
...
LoadModule userdir_module libexec/apache24/mod_userdir.so
...
# User home directories
Include etc/apache24/extra/httpd-userdir.conf
...
```
Next we want to make sure that **PHP** scripts get executed. For that we need to create a configuration file.   
Fire up your editor of chocie and create `/usr/local/etc/apache24/modules.d/001_php.conf` with the following contents.
```bash
<IfModule dir_module>
        DirectoryIndex index.php index.html
        <FilesMatch "\.php$">
                SetHandler application/x-httpd-php
        </FilesMatch>
        <FilesMatch "\.phps$">
                SetHandler application/x-httpd-php-source
        </FilesMatch>
</IfModule>
```
This snippet will make it possible to execute PHP scripts.   

**Reload** the **webserver** to activate the configuration we've just done.
```bash
$ sudo service apache24 reload
```

Alright, let's head over to configuring **PHP**!

# PHP fun
First of all: Let's copy over the sample **php.ini**.
```bash
$ sudo cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini
```
Next, edit the just copied config.
```bash
[Date]
...
; Defines the default timezone used by the date functions
; https://php.net/date.timezone
date.timezone = "Europe/Berlin"
...
```
The only change I've done here is setting the correct **timezone**.    
Feel free to change whatever you need! For me, this is all I need for now.

Now, enable and start the PHP-FPM service.
```bash
sudo sysrc php_fpm_enable="YES"
sudo service php_fpm start
```
Create a test **PHP** file under `public_html` with the following content - The file being called `test.php` (Amazing name, I know).
```bash
<?php

phpinfo();
```
Be sure to also make the file **executable**!
```bash
$ chmod +x public_html/test.php
```

After all that is done, restart **Apache** and check if the just created **PHP** file executes correctly.
```bash

$ sudo service apach24 restart
$ firefox http://vfbsd/~x/test.php
```
You should see the output of `phpinfo()` in your **browser**.   
**If not**: Double check that everything is installed and configured!

# Adding Python to the mix
First things first, we need to install **Python**.
```bash
$ sudo pkg install python
```
To make it possible for **Apache** to execute **Python** scripts we need to enable `mod_cgi`. This is done via editing `http.conf`.
```bash
$ sudo vim /usr/local/etc/apache24/httpd.conf
```
```bash
...
<IfModule mpm_prefork_module>
        LoadModule cgi_module libexec/apache24/mod_cgi.so
</IfModule>
...
```
Uncomment the line shown above and we are all set!

Next we have to edit the UserDir configuration file a last time to make it all work.
Open `/usr/local/etc/apache24/extra/httpd-userdir.conf` with your editor of choice and alter the configuration as shown.
```bash
<Directory "/home/*/public_html">
    AllowOverride FileInfo AuthConfig Limit Indexes
    Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec ExecCGI
    AddHandler cgi-script .cgi .py 
    Require method GET POST OPTIONS
</Directory>
```
Make sure to add `ExecCGI` and the `.py` extension!

After that, reload **Apache**.
```bash
$ sudo service apache24 reload
```
The only thing left to do is to create a sample **Python** script to test if everything is working.
Again, use your editor of choice and create `test.py` - Or any name you like to give said script.
```bash
#!/usr/local/bin/python

print ("Content-type:text/html\r\n\r\n")
print ('<html>')
print ('<head>')
print ('<title>Testing</title>')
print ('</head>')
print ('<body>')
print ('<h1>Python Testing Page</h2>')
print ('</body>')
print ('</html>')
```
Let's mark the script **executable**.
```bash
$ chmod +x public_html/test.py
```

All done, let's open the script in our **browser**.
```bash
$ firefox http://vfbsd/~x/test.py
```
**It works (TM)**!   

# Final words
Of course, this is very much just a little appetizer for what **Apache**/**PHP**/**Python** can do. **Apache** alone is very versatile and can do many more things - be sure to check out the **documentation** / **MAN pages**.

All in all - that is all that I needed.
A simple way to create **PHP** / **Python** scripts in a directory in my **$HOME**.
A simple setup for a simple coder.

**Enjoy**!

...and as always:

**Stay Open!**