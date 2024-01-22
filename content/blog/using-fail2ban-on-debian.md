+++
title = "Using Fail2ban on Debian"
date = "2023-11-05T14:10:23+01:00"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."

tags = ["debian","fail2ban",]
+++

# Installing Fail2Ban
Fail2Ban is available in the Debian repository.
You can install it as follows:

```shell
$ sudo apt install fail2ban
```

# That's it - kinda...
By default there are no logs to read from. Rsyslog was removed in Debian 12, which means there are no logfiles to read for Fail2Ban.
However, Fail2Ban can read the journal log provided with systemd (Journalctl).

# Leveraging the journal
First, create a local config file for Fail2Ban:

```shell
$ sudo touch /etc/fail2ban/jail.local
```

Open the file with your favorite editor:

```shell
$ sudo vim /etc/fail2ban/jail.local
```

Paste in the following content:

```ini
[sshd]
enabled   = true
backend   = systemd
port      = 22
maxretry  = 3
findtime  = 10m
bantime   = 30d
ignoreip  = 127.0.0.0/8
```

The important setting here is "backend = systemd".
This will advise Fail2Ban to consult the journal.

Make sure to restart/reload Fail2Ban after making the modifications:

```shell
$ sudo systemctl restart fail2ban
```

# Final notes
You need to adapt the settings given in jail.local to your needs.
This example only targets sshd (OpenSSH).

Seeing which jails are active:
```shell
$ sudo fail2ban-client status
```

That's it. Have fun everyone!
