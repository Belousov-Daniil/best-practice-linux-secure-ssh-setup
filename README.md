# Remote server setup essentials

First remote server connection

```bash
ssh root@server-ip
```

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
sudo adduser user-name
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
Open `/etc/ssh/sshd_config` with something like `nano` (!with sudo ofc!) and make sure that this lines are exist and not commented.

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

That's it, now your server has bare essential secure policy.

### Additionals

This repo is also contains some useful scripts to solve daily tasks, like, how do I
list all users on my server and etc. Usually, name of a task - name of a sh script within this repo.
