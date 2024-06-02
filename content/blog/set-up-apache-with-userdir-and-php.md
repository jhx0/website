+++
title = "Set up Apache with userdir and PHP"
date = "2024-06-02T13:54:11+02:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["linux", "apache", "webserver", "userdir", "php", "debian"]
+++

# Why?
I wanted to setup a **webserver** on my local system with **PHP** enabled to do some fiddling around.   
The process on **Debian** is quite simple... so simple that you just need to install the packages and everything will already work - given you put your **PHP** script files under **/var/www/html**.
But in this case I wanted to enable the **userdir** option with the possibility to run **PHP** scripts from it.   
 
So, let's get **right to it**!

# Installing needed packages
Using **Apt**, all the packages are readily available for us in the **Debian** repositories.

```shell
$ sudo apt install apache2 php libapache2-mod-php -y
```

*Done*. The next step will be setting up **user directory**.

# Userdir setup
All we basically need to do here is creating the directory **"public_html"** on our home directory.   

```shell
$ mkdir ~/public_html
```

**Note:** *This can be changed*, if one so desires. Have a look at the following **config** file:

> /etc/apache2/mods-enabled/userdir.conf

For **Apache** being able to access the directory we just created, we need to make sure that there are execute **"x"** permissions on the parent directory.   
**Fixing** this is simple:

```shell
sudo chmod +x /home/USER/
```

**Attention:** This **might** be a **security issue**!   
Think about this before blindly copy pasting the command*. You can always change the location and name of the userdir! (**See note above**)

**Finally** we need to make sure the **userdir** module is loaded.

```shell
$ sudo a2enmod userdir && sudo systemctl reload apache2
```

Now, on to the **next step**.

# Configuring PHP
By default, **PHP** is not enabled when leveraging the **userdir module**. To change this behaviour we need to edit a **config** file.

```shell
sudo vim /etc/apache2/mods-available/php8.2.conf
```

A **snippet** from the given file:

```shell
# Running PHP scripts in user directories is disabled by default  
#  
# To re-enable PHP in user directories comment the following lines  
# (from <IfModule ...> to </IfModule>.) Do NOT set it to On as it  
# prevents .htaccess files from disabling it.  
<IfModule mod_userdir.c>  
     <Directory /home/*/public_html>  
         php_admin_flag engine Off  
     </Directory>  
</IfModule>  
```

**Comment** the entire **IfModule** block like shown below.

```shell
#<IfModule mod_userdir.c>                                                        
#     <Directory /home/*/public_html>                                            
#         php_admin_flag engine Off                                              
#     </Directory>                                                               
#</IfModule>
```

After altering the **config**, make sure to reload **Apache** so the changes are taking effect.

```shell
$ sudo systemctl reload apache2
```

# Creating a test script
Anyone familiar with **PHP** has there own way to test if script execution is working.   
I personally like to call *phpinfo()* to see if all is well.

Create a new ***.php** file in the directory:

```shell
$ touch ~/public_html/test.php
```

The content of my **file** will be as follows for example.

```php
<?php

phpinfo();
```

With this all done it is time to *test if everything works*.

# Testing out PHP
It is finally time to see if **PHP** scripts can indeed be executed.
Simply pull up your favorite browser and navigate to the given **URL**:

> http://localhost/~USER/test.php

**Note:** Make sure to use *your username* instead of *USER*.

If everything is **right** you will see a similar **output** as I do.

![PHP](/images/php-userdir.png)

# Closing words
Setting up **Apache** with **PHP** certainly is not a hard task. **Debian** makes this quite easy.   
*Depending on your needs* this might be all there is to it.    

**EOF** ...for now.

**Stay Open!**
