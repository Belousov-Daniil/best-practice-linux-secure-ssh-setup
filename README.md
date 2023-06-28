# Remote ssh setup on Linux essentials

    ███████╗███████╗██╗  ██╗
    ██╔════╝██╔════╝██║  ██║
    ███████╗███████╗███████║
    ╚════██║╚════██║██╔══██║
    ███████║███████║██║  ██║
    ╚══════╝╚══════╝╚═╝  ╚═╝

First server connection

```bash
ssh root@server-ip
```

To not enter this each time, just create an alias for your shell like `alias server-name=ssh your-user@your-server-ip` withing config file.
For bash by default this file is `~/.bashrc` or `~/.bash_profile`.

After that you will be prompted for root password.
Then, you will be connected to remote server terminal session.

### Change 'default' user

Before any additional installations and setup you first need to secure your server, preventing
any unauthorized connections. To do that, create a new user.

Why? Cause intruder's bots are searching web for any public through the Web and trying to connect
via ssh with the `root` user by default at every moment of time. You can lookup at ssh logs 
on your server to be sure about this.

To prevent this, login for a root use must be prohibited, so the intruders or bots would be unable
to connect to your server automatically or manually, if you choose a non-standart 'default' username.
So the first thing you need to do is create a new user with non-standart name and add it to a `sudo` group,
because you will use it as a 'default' user.

```bash
# Add new user to OS
sudo adduser user-name

# Add user to OS to sudo group
sudo usermod -aG sudo user-name
```

Then, you be prompted for user password and other non-neccesary creditals, which allowed to be skipped.
Exit server with exit command.

### Change ssh policies

Now you need to copy your ssh key to connect to a server without password. Create a ssh key pair,
if don't have it.

```bash
ssh-copy-id user-name@server-ip
# Then enter user password
```

Connect via a new user without password. All you have to do now is change ssh config and reload ssh service.
Open `/etc/ssh/sshd_config` with something like `nano` (!with sudo ofc!) and make sure that this lines are exist and not commeted.

```txt
PasswordAuthentication no
PermitEmptyPasswords no
PermitRootLogin no
ChallengeResponseAuthentication no
```

```bash
# ssh is a ssh service client/interface
sudo service ssh restart

# sshd is a ssh service server
sudo service sshd restart
```

### Prevent brute-force

This step is solving by enabling any kind of system daemon which will
lookup for network suspicious activity on machine and prevent potentials brute-force attacks.

Using fail2ban for this purpose is highly recomended due its stability and community support,
it's also very easy to maintain and configure, and you get it ready-to-use out of box.

    ███████╗ █████╗ ██╗██╗     ██████╗ ██████╗  █████╗ ███╗   ██╗
    ██╔════╝██╔══██╗██║██║     ╚════██╗██╔══██╗██╔══██╗████╗  ██║
    █████╗  ███████║██║██║      █████╔╝██████╔╝███████║██╔██╗ ██║
    ██╔══╝  ██╔══██║██║██║     ██╔═══╝ ██╔══██╗██╔══██║██║╚██╗██║
    ██║     ██║  ██║██║███████╗███████╗██████╔╝██║  ██║██║ ╚████║
    ╚═╝     ╚═╝  ╚═╝╚═╝╚══════╝╚══════╝╚═════╝ ╚═╝  ╚═╝╚═╝  ╚═══╝

> *Daemon to ban hosts that cause multiple authentication errors*
> [Official website](https://www.fail2ban.org/) - [Github](https://github.com/fail2ban/fail2ban)


```bash
# Installation
sudo apt install fail2ban
```

Although it comes preconfigured through `/etc/fail2ban/jail.conf` and you can configure this tool by changing this file, it's highly not recommended.
All configuration must be done with separate .conf files within `/etc/fail2ban/jail.d/` directory, and that's why:

- Secure system - is always a 'fresh' system, so you should/will periodically do something like `apt update`. But, if something comes preconfigured, then it's default files, and so the configuration, can be overwrittenm, changed or removed due to updates, **SO** it not just a problem to recover configuration, but it can turn off security policies on machine, especially when you don't expect this.

- Configuration through one file is not maintain-friendly and it's quite difficult to automate such configurations, cause you will need to implement not just a tool to create/remove/change separate confing files, but also to inspect one large, barely readable file.

Configuration is done with [INI](https://en.wikipedia.org/wiki/INI_file) config format.

Write to file: `/etc/fail2ban/jail.d/ssh-service-jail.conf`

```ini
[ssh]
enabled = true
port = ssh
findtime = 3600
maxretry = 3
bantime = 86400
filter = sshd
logpath = /var/log/auth.log
```

Write to file: `/etc/fail2ban/jail.d/sshd-service-jail.conf`

```ini
[sshd]
enabled = true
port = ssh
findtime = 3600
maxretry = 3
bantime = 86400
filter = sshd
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

Write to file: `/etc/fail2ban/jail.d/defaults-debian.conf`

```ini
[DEFAULT]
igonreip = 127.0.0.1
```

> Do not write findtime, maxretry, bantime to `default` statement, because it may cause different
> problems, undefined or conflicting behaviour in other services or policies because of the fact that default config
> probably may contain already enabled policies

```bash
# Restart service
sudo systemctl restart fail2ban
```

Useful commands

```bash
# Check jails config
sudo fail2ban-client status

# Check bantime for service - it's a name if brackets you write to .conf file
# For example for service 'test-123' you should use
sudo fail2ban-client get test-123 banip --with-time

# Get fail2ban logs
sudo zgrep 'Ban' /var/log/fail2ban.log*
```

That's it, now your server has bare essential secure policy for ssh.

### Additionals

This repo will also will fill up with some useful scripts to solve daily tasks, like, how do I
list all users on my server and etc, that can be associated with repo topic.

Usually, name of a task - name of a script within this repo.