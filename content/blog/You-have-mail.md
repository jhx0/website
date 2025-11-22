+++
title = "You Have Mail"
date = "2025-11-22T15:10:39+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["bsd", "openbsd", "freebsd", "unix", "mail", "source", "src", "code", "programming"]
+++

If you've ever logged in to a **FreeBSD** system, you have certainly seen the line **"you have mail"**.   
No big deal might one say, it simply tells the user that there is mail to be read.   
Indeed, that is the case... but have you maybe wondered how that is possible?   
After all, it must come from somewhere, **right**?!

# Where to begin?
First of all, there must be some code that generates that message.
Given that the message is displayed after logging in to the system, the culprit must be **login(1)**.   
This very program is what is spawned when you log onto your **FreeBSD** system - it also does the important job of authorizing the given user (Would be quite a mess if anyone could logon to the system after all - esp. **without a password**!)

# Where to look?
Well, the only way we can find out where this message is coming from is checking the **source code** for the login program.   
With all that said it is time to do just that!

# Getting the src
If during install of your **FreeBSD** system you've slected **"src"** than you are already set up and the sources can be found under **/usr/src**.   
If you don't have the **FreeBSD** src tree you have multiple options available.   
You can either **clone** the src tree or download a tarball and extract it.   
Information can be found under the following link: https://docs.freebsd.org/en/books/handbook/mirrors/   
If you want to follow along without getting the src tree you can use the public **GitHub mirror** - which makes it easy to just browse the src tree in the browser.   
**GitHub mirror:** https://github.com/freebsd/freebsd-src

Alright, we are all set, let's **dig into the code**!

# Tell me your secret
We have already established that during login a program named **"login"** is spawned on the **tty**.   
Doing a `which login` tells us that the program is located under **"/usr/bin/login"**.   
Given this clue let's look at the src tree.   
If we navigate to **src/usr.bin** we can find a directory named **"login"** - this is the very source code of the program we are looking for!   
In the directory in question we find the file **login.c** - A C program, where all the magic happens.   
Let's open the file in our editor of choice and **dig into it**.

```shell
$ vim /usr/src/usr.bin/login/login.c
```

So far so good... but were is the **magic** that makes the message appear on login?   
We are going to **find out soon**!

# Delving into the code
Let's navigate to line **609** first.

```C
    if (!quietlog) {
        ...
```
Here we can see that a so called **flag** is checked (This is just a variable in C).   
**quietlog** simply indicates if **"hushlogin"** is set by the user - in which case nothing gets printed to the screen! (**Keep that in mind**)

Let's **move on** to the meat and potatoes of the code in question (The listing will be a little longer).
```C
        const char *cw;

		cw = login_getcapstr(lc, "welcome", NULL, NULL);
		if (cw != NULL && access(cw, F_OK) == 0)
			motd(cw);
		else
			motd(_PATH_MOTDFILE);

		if (login_getcapbool(lc_user, "nocheckmail", 0) == 0 &&
		    login_getcapbool(lc, "nocheckmail", 0) == 0) {
			char *cx;

			/* $MAIL may have been set by class. */
			cx = getenv("MAIL");
			if (cx == NULL) {
				asprintf(&cx, "%s/%s",
				    _PATH_MAILDIR, pwd->pw_name);
			}
			if (cx && stat(cx, &st) == 0 && st.st_size != 0)
				(void)printf("You have %smail.\n",
				    (st.st_mtime > st.st_atime) ? "new " : "");
			if (getenv("MAIL") == NULL)
				free(cx);
		}
```
**What happens here**?
Well, simply put:
- The code checks for a capability named welcome - Concering the **MOTD** (**Message of the Day**)
- Also, it checks if **"nocheckmail"** is set - in which case there will be no mail checkup at all!
- The bottom part deals with the actual mail check - You can see the string **"You have %smail"** there

Let's take a **closer look** at the actual mail check:

```C
            cx = getenv("MAIL");
			if (cx == NULL) {
				asprintf(&cx, "%s/%s",
				    _PATH_MAILDIR, pwd->pw_name);
			}
			if (cx && stat(cx, &st) == 0 && st.st_size != 0)
				(void)printf("You have %smail.\n",
				    (st.st_mtime > st.st_atime) ? "new " : "");
			if (getenv("MAIL") == NULL)
				free(cx);
```
Let me explain a little here:
First, **getenv** (**libc** function) is called with the env variable name **MAIL**.   
It is checked if it contains **NULL** - which would mean the variable is not set.   
The second if construct is where the magic happens. You can see that there is a call to **stat()**.   
**stat()** is a function that returns information about a file like **size**, **access time**, etc. In this case, **st_size** is checked.   
Now I need to **explain** a little more...   
In the beginning there was a line that stated:
```C
	cx = getenv("MAIL");
```
This very line assigns the content of **MAIL** to the variable **cx**!
This would in turn mean that **cx** could contain for example a string like **"/var/mail/user"**.   
Since stat is called on that path, **stat(cx, &st)**, the information about the given file is now located in the structure/struct (A C data structure) **"st"**. Hence, we can access the size of the file in C via **st.st_size** - This gives us the size of the file in question in **bytes**.

```C
st.st_size != 0
```
As long as the size of the file is **not zero**, which would mean it is empty, the evaluation returns true and the following statment is run.
```C
(void)printf("You have %smail.\n",
				    (st.st_mtime > st.st_atime) ? "new " : "");
```
Basically, **st.st_mtime** and **st.st_atime** are compared - **access**/**modification times** of the given file. (This code checks if there are any changes).   
But wait... it says **"You have %smail"**? That is not printed on my screen!   
Yes, that is **right**!   
...There is a little more **magic** to it.

To make it simple. **printf()** prints text to a screen in C. The placeholder (Format string) **"%s"** stands for string.   
Let's make a long story short at this point and evaluate the last secret this call to **printf()** offers us.
```C
...
(st.st_mtime > st.st_atime) ? "new " : "")
```
This small little statement decides wheter it prints **"You have new mail"** or **"You have mail"**.   
The deciding factor here is if the access time of the file in question is newer or older - depending on the case it either returns **"new"** or simply a empty string **""**.

And this is where the **You have mail** line comes from!

# Digging a little further...
Now the question that pops up in my mind is: **Was this always done like this?**   
Well, **good question**!   
At first I wanted to check **OpenBSD** and see if the mail notification is handled the same way.   
Good thing there is a **GitHub mirror** to make our lifes a little easier:   
https://github.com/openbsd/src   

The code for login can be found under **src/usr.bin/login/login.c** - if anyone wants to follow along.

Navigating to line **685** shows us the following:
```C
if (!quietlog) {
		if ((copyright =
		    login_getcapstr(lc, "copyright", NULL, NULL)) != NULL)
			auth_cat(copyright);
		motd();
		if (stat(mail, &st) == 0 && st.st_size != 0)
			(void)printf("You have %smail.\n",
			    (st.st_mtime > st.st_atime) ? "new " : "");
	}
```
And yes indeed, the message itself is handled the **exact same way** as it is done in **FreeBSD**!

We could stop here and just be happy that we've learned something... but I had to take a look at a **older source tree**, to be exact, the sources for **4.4BSD-Lite2**.   
We can find the src tree online on **GitHub** as well:   
https://github.com/dank101/4.4BSD-Lite2   
Under **usr.bin/login/login.c** we find what we are looking for.
```C
if (!quietlog) {
		(void)printf("%s\n\t%s  %s\n\n",
	    "Copyright (c) 1980, 1983, 1986, 1988, 1990, 1991, 1993, 1994",
		    "The Regents of the University of California. ",
		    "All rights reserved.");
		motd();
		(void)snprintf(tbuf,
		    sizeof(tbuf), "%s/%s", _PATH_MAILDIR, pwd->pw_name);
		if (stat(tbuf, &st) == 0 && st.st_size != 0)
			(void)printf("You have %smail.\n",
			    (st.st_mtime > st.st_atime) ? "new " : "");
	}
```
**Same game**!
This means that the code for handling the mail notification has not changed in many many years given that the **4.4BSD-Lite2** code features the same logic!

Well, **we live and learn**.

Ok, so far so good. But: There is one more thing I want to check. I want to see if the latest **UNIX** did feature the same logic. So i took a look at the sources from **Unix Tenth Edition (1989)** just to make sure.   
The code for **login.c** can be found here:   
https://minnie.tuhs.org/cgi-bin/utree.pl?file=V10/cmd/login.c
```C
if (cmd==NULL) {
		showmotd(motd);
		strcat(maildir, pwd->pw_name);
		if(access(maildir,4)==0) {
			struct stat statb;
			stat(maildir, &statb);
			if (statb.st_size > POSTMKSIZ)
				printf("You have mail.\n");
		}
	}
```
And the code is different from the **BSD variants** we've looked at.
This leads me to believe that the code in question first surfaced in the **BSD sources**.   
Again, **something learned**!

# The end... for now
Sometimes a **little discourse** in the **src** can be quite enlightening and fun!   
Depending on ones skill level, anything can be changed/altered - and there are many things to discover/uncover.   
The **truth** lies always burried in **source code** - no matter which programming language is leveraged!

I don't claim that I've got all the details done perfectly. I make mistakes just like anyone else.   
In the end, it is simply fun to delve into code that makes up everything around us.

Until next time...

**Stay Open!**
