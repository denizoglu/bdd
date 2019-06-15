# What to Do First When You First Install a Brand New Server

[TOC]

# Table of Contents
1. [Change Password of the `root` Account ASAP](#1-change-password-of-the-root-account-asap)
2. [Update and Upgrade the Programs ASAP](#2-update-and-upgrade-the-programs-asap)
3. [Install fail2ban](#3-install-fail2ban)
4. [Create a new power user and add it to the relevant access groups](#example4)



So, I've bought a new VPS for my project on Hetzner and I've installed Ubuntu 18.04 TLS version.

The IP address I got for this server is 116.203.189.123. Therefore I'd like to connect to my server through SSH:

```bash
sd@sd-REDUNIX:~$ ssh root@116.203.189
```

Here my local computer name is `sd-REDUNIX`, and my username is `sd`.

If you haven't added your SSH key, you'll be asked with the `root` account password. Enter your password if it is provided you thorough e-mail, otherwise if your server doesn't set up one, it will directly grant access and you'll get this:

```bash
sd@sd-REDUNIX:~$ ssh root@116.203.189.123
The authenticity of host '116.203.189.123 (116.203.189.123)' can't be established.
ECDSA key fingerprint is SHA256:9OTDDHiK4B9IvhSXag+FEECrak9tXm88BbdynAH+7ws.
Are you sure you want to continue connecting (yes/no)?
```
Type `yes` and hit Enter.

You'll have:
```bash
sd@sd-REDUNIX:~$ ssh root@116.203.189.123
The authenticity of host '116.203.189.123 (116.203.189.123)' can't be established.
ECDSA key fingerprint is SHA256:9OTDDHiK4B9IvhSXag+FEECrak9tXm88BbdynAH+7ws.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '116.203.189.123' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-50-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Jun 14 12:49:29 CEST 2019

  System load:  0.0               Processes:           94
  Usage of /:   4.2% of 37.50GB   Users logged in:     0
  Memory usage: 3%                IP address for eth0: 116.203.189.123
  Swap usage:   0%


54 packages can be updated.
25 updates are security updates.
```

Most possibly, there will be updates available as above. But before ALL, we need to setup a good password for the `root` account.

# 1. Change Password of the `root` Account ASAP

Enter a preferably long, easy to remember password for yourself twice:

```bash
root@rubi-beta:~# passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

I strongly advise you to record it in a good password manager like [KeePass](https://keepass.info/). Even better, install KeePass first, and create a very strong password with it. You can find more information on how to install KeePass for your system and synchronize your password database across multiple computers _-and even mobile phones-_ **[here](https://www.howtogeek.com/165882/how-to-use-keepass-in-your-browser-across-your-computers-and-on-your-phone/)**.

Now we have to a update apt-get database and then order it to install all updates available:

# 2. Update and Upgrade the Programs ASAP

```bash
root@rubi-beta:~# apt-get update
root@rubi-beta:~# apt-get upgrade
```

It will most probably ask you for your input on some critical updates and what course of action to take. Reply accordingly to finish the update/upgrade process.

# 3. Install **`fail2ban`**

One of the first and foremost security precautions to be taken (_IMHO_) is to install **`fail2ban`**, which is a daemon that monitors incoming login requests and blocks suspicious activity by blacklisting IP addresses that apparently are not with good will.

```bash
root@rubi-beta:~# apt-get install fail2ban
```

You are strongly recommended to install it and see the logs for yourself just to see once again that Internet is definitely a safe space.

# 4. Create a new power user and add it to the relevant access groups

After these steps, it is a safe practice to create a new user that will have elevated access rights to carry on with installation and deployment, rather than using the root account itself.

For this, we'll create a so-called _admin_ user (`sd`). For this, we will use `adduser` command:

```bash
root@rubi-beta:~# adduser sd
Adding user `sd' ...
Adding new group `sd' (1000) ...
Adding new user `sd' (1000) with group `sd' ...
Creating home directory `/home/sd' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for sd
Enter the new value, or press ENTER for the default
	Full Name []: Sefa Denizoglu
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] Y
```

Now we need to grant him `sudo` access by adding him/her to the `sudo` group, as well as to the `wwww-data` group for web development and deployment purposes:

```bash
root@rubi-beta:~# usermod -aG sudo,www-data sd
```

# 5. Set up SSH and Disable Password Authentication

## Set up SSH

We need to create .ssh directory in the newly created user's home directory,, and edit access rights accordingly, to let us ssh into the remote web server:

```bash
root@rubi-beta:/home/sd# mkdir /home/sd/.ssh && chmod 700 /home/sd/.ssh

```

Let us now the remote web server know to whom it can grant access. To do this, we need to create and edit an `authorized_keys` file under .ssh directory:

```bash
root@rubicon:~# vim /home/sd/.ssh/authorized_keys
```

From your own (local) computer, you'll have to copy the content of your own public key (found at `/home/your_username/.ssh/id_rsa.pub`) and paste into that file in the remote server (`/home/sd/.ssh/authorized_keys`). Save and quit afterwards. To this file, you should add public keys of any other machine that you'd like enable to ssh into the remote server.

We now have to change the access rights of so that _only the owner_ (in this case, user `sd`) can _read_ the file. The file gets read-only even for the owner. It's just a precaution to prevent accidental editing of the file. And then we change the owner of the whole `/home/sd` directory to `sd` recursively, that is, all files and folders under this directory are from now on owned by `sd`.

```bash
root@rubicon:~# chmod 400 /home/sd/.ssh/authorized_keys
root@rubicon:~# chown sd:sd /home/sd -R
```

## Test Access Rights
> **Keeping a root login session open (as you'd not want to lock yourself out of the server)**, let us test the new user:

```bash
sd@sd-REDUNIX:~$ ssh sd@rubiconmedya.com
```

If you had not set up a passphrase for your private key (on your local computer), then you will be logged in directly as follows. Otherwwise, you'll be asked for it. Enter you passphrase to unlock your private key and log in to remote server:

```bash
sd@sd-REDUNIX:~$ ssh sd@rubiconmedya.com
The authenticity of host 'rubiconmedya.com (116.203.189.123)' can't be established.
ECDSA key fingerprint is SHA256:vitaSKixALW09AkEAHCRZ1Hg61JRZH6Tu3RdNKfFe58.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'rubiconmedya.com,116.203.189.123' (ECDSA) to the list of known hosts.
```

## Disable Password Authentication 

# 6. Set up Firewall and Tune for Access



# 7. 
