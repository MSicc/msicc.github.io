---
id: 6361
title: 'Create an additional SSH-login enabled user for your Azure Linux VM without third-party tools'
date: '2019-09-12T17:30:20+02:00'
author: 'Marco Siccardi'
excerpt: 'In this post, I am going to show you how to create a new user and enable login only via SSH for your Linux VM on Azure without any third-party tools.'
layout: post
permalink: /create-an-additional-ssh-login-enabled-user-for-your-azure-linux-vm-without-third-party-tools/
image: /assets/img/2019/09/cyberspace-2784907_1280.jpg
categories:
    - Azure
    - Linux
tags:
    - Azure
    - chgrp
    - groupadd
    - Groups
    - keypair
    - Linux
    - OpenSSH
    - password
    - 'private key'
    - 'public key'
    - SSH
    - ssh-keygen
    - sshd_config
    - user
    - useradd
    - usermod
---

As I am moving forward in my current Linux journey, I recently came into a situation where a second user would have been handy. So I tried a few things to create the new user and allow the new user to only log in via SSH.

### What the h\*\*\* is SSH?

SSH stands for Secure Shell and describes a protocol to connect via encrypted credentials. The security is provided by cryptographic keys, where the server only knows the public key and the client that wants to connect needs the matching private key. The most popular implementation is [OpenSSH](https://www.openssh.com/), which is available as an additional feature on Windows 10 since last fall. If you want to learn more about SSH, read the [Wikipedia entry](https://en.wikipedia.org/wiki/Secure_Shell) as well.

### Create the SSH key pair on Windows

#### Install the OpenSSH client 

First, install the OpenSSH client on your Windows machine. Proceed as follows:

- open the start menu and type ‘apps’
- select ‘Apps and features’ settings page
- select ‘Optional features’
- click on ‘Add a feature’
- search ‘OpenSSH client’, click on it and ‘Install’

If you are scrolling down the list of installed features, you should find the entry for the OpenSSH client.

![](/assets/img/2019/09/openssh-client-feature-installed.png)
*Note: If you are not able to install this feature, it may be the right time to update your Windows installation to the latest version.*

#### Create your SSH key pair

If your user profile does not have a folder called ‘.ssh’, it is time to create it now. Type ‘`%USERPROFILE%`‘ in the Windows Explorer’s address bar to get to the right folder immediately and create the folder.

Now that we have the folder OpenSSH searches for, we are already able to create our new SSH keypair. Open a command prompt and type this:

``` shell
 ssh-keygen -t rsa -b 4096 -C "newuser@machine.com"
```
 
This will initiate the keypair creation. The `-C` parameter is optional and can be anything. You’ll find these keys often created with `<user at address>` combinations. After OpenSSH has created your keypair in memory, it will ask for a location to save the file. If you do not enter anything, it’ll save it as `id_rsa` into the .ssh folder created earlier. If you want to save it to another file name, you can do so:

``` shell
 C:\Users\username\.ssh/id_test
```
 
Please note that the file name (without any extension!) is separated by ‘/’, not by ‘\\’. If you do not respect this, it will give you an error that the file name does not exist. This happens only if don’t use the default filename. After the files are created, OpenSSH asks you for a passphrase to protect your private key. Nowadays, every single keypair should be password protected (just my 2cts). Once the creation is complete, you’ll see something like this:

``` shell
 Your identification has been saved in C:\Users\msicc\.ssh/id_test.
Your public key has been saved in C:\Users\msicc\.ssh/id_test.pub.
The key fingerprint is:
SHA256:8LK33fWKXMkY5FtcN4uU1v9SyCSUqKNp2T/PurpZCRU newuser@machine.com
The key's randomart image is:
+---[RSA 4096]----+
|          E...   |
|          .o. o  |
|      .  .. o+.oo|
|       oo. oo=.o=|
|      .=S.  o.=.o|
|      =o.. . * o.|
|     .. ..o o * .|
|       . =o+ + o |
|        =o+=* ...|
+----[SHA256]-----+
```
 
As you can see, we are able to create SSH keys without any third-party application on Windows. You can now safely close the console window (e.g by typing ‘`exit`‘).

### Deploying the public key to the server

Of course, the whole key pair thing has only sense if we are using it to secure our client/server communication. On Linux, there would be the easy to use command `ssh-copy-id` to deploy the key. Some Windows tutorials are showing the `scp` command, but I never got that working with my Azure VM. The only way left was to deploy it manually (which is not that difficult if you know how).

#### The manual way

After logging in (using Azure CLI), we are going to add a new user to the gang:

``` shell
 sudo useradd -m newuser
```
 
This will create a new user and its home directory. If the OS asks you for a password, it is up to you and the OS settings for empty passwords to provide one or not. Next step would be to add it to one or more groups if necessary:

``` shell
 sudo usermod -aG sudo newuser
```
 
The -aG parameter of the `usermod `command adds the specified group(s) to the user’s groups table. If you want to add the user to more than one group, separate them with a comma (no whitespace after the comma!).

To make things a little bit easier for us, we are going to log in as the new user:

``` shell
 sudo su newuser
```
 
*Note: If you want to proceed without logging in as the new user, you’ll need to change* `~` *to* `/home/newuser` *for the following commands.*

To make Linux accept our prior created SSH key only for our new user, we need to create the .ssh folder and the file for allowed keys in the new user’s home directory. Let’s start with the .ssh directory:

``` shell
 sudo mkdir ~/.ssh
sudo chmod 0700 ~/.ssh
sudo chown newuser:newuser ~/.ssh
```
 
Let’s break it down. Obviously, the `mkdir` command creates the .ssh folder. The `chmod `command with the` 0700` parameter gives full access only to our new user. Finally, the `chown` command makes our new user the owner of the file.

Linux saves the allowed keys in a file called ‘`authorized_keys`‘, so that’s the next we are going to create. After creation, we change the file’s access rights to `0644`, which makes it read-only for all users except our new user. Execute these commands:

``` shell
 sudo touch ~/.ssh/authorized_keys
sudo chmod 0644 ~/.ssh/authorized_keys
```
 
We’re getting closer… earlier, OpenSSH created two files in the .ssh folder on Windows. We are going to copy the contents of the .pub file into the authorized\_keys file now. You can extract the content of the .pub file with Notepad on Windows. Once you have that one in your clipboard, open the authorized\_keys file (using your favorite editor):

``` shell
 nano ~/.ssh/authorized_keys
```
 
Paste the content of your .pub file by right-clicking on the Azure CLI window. Save the file and close it. To secure the new user account, we need to apply some additional steps.

The first one is to delete and disable the password of our new user:

``` shell
 sudo passwd -d -l newuser
```
 
The `-d` parameter completely deletes the password. The `-l` parameter locks the state, preventing the user to set a new password (without using `sudo`, that is).

#### Additional security measures

Now that we have our SSH public key on our server, we are save to disable the password-based login in general. To do so, we need to modify the `sshd_config` file of our server. Open it with the editor of your choice:

``` shell
 sudo nano /etc/ssh/sshd_config
```
 
Search for `PasswordAuthentication `and `ChallengeResponseAuthentication`. Uncomment these entries if necessary by removing the ‘`#`‘ in front of the line an set them to ‘`no`‘. Some tutorials that are floating around the web tell you to even set `UsePAM` to no, but following this recommendation always disabled the login completely for me on the Azure VM and I always had to reset the SSH config via the Azure CLI.

Save and exit the `sshd_config` file. To take these changes into effect, we need to restart the SSH service on our service:

``` shell
 sudo service ssh restart
```
 
Wait a few seconds and then verify that the service restarted by controlling its status:

``` shell
 systemctl status ssh.service
```
 
That’s it, we have deployed our new SSH key to our server and took additional security measures. There’s just one thing left: try to log in via ssh as the new user. This is a pretty easy task. In the Azure CLI (or a local command prompt), enter the following command:

``` shell
  ssh -i %USERPROFILE%\.ssh\id_test newuser@machine.com
```
 
After entering your password, you should be logged in just like you did with your admin user.

### Bonus: add a shared directory

More often than not, you might want to create scripts or other files that are available to all users. Follow these simple steps:

``` shell
 sudo groupadd shared
sudo usermod -aG shared newuser

sudo mkdir -p /var/helpers
sudo chgrp -R shared /var/helpers
sudo chmod -R 2775 /var/helpers
```
 
The first two lines create a new group and assign our new user to the group. Repeat the second command for every user you want to be in that group.

Then we create the shared folder `/helpers` in the `/var` folder of our server. Utilizing the `chgrp` command, we give the ownership to our shared group. Last but not least, we are modifying the access rights once again for the `/var/helper` folder. The combination `2775` means that every new file inherits the group from the folder, allowing all members of the group to read, write and execute the file, while users outside the shared group only can read and execute the file.

### Conclusion

As you can see, one does not always have to use third-party tools to get things done. Like I said at the beginning of this post, once you know the steps that are needed, it is pretty easy to create a new SSH key pair, create a new user and manually deploy the public key to the server. As always, I hope this post is helpful for some of you.

#### Helpful links

- <https://help.ubuntu.com/lts/serverguide/openssh-server.html>
- <https://manpages.ubuntu.com/manpages/trusty/man8/groupadd.8.html>
- <https://manpages.ubuntu.com/manpages/xenial/man1/chgrp.1.html>
- <https://manpages.ubuntu.com/manpages/xenial/man8/usermod.8.html>
- <https://cheatsheetworld.com/programming/unix-linux-cheat-sheet/>
- <https://help.ubuntu.com/community/LinuxFilesystemTreeOverview>

[Title Image Credit (Pixabay)](https://pixabay.com/photos/cyberspace-data-wire-electronic-2784907/)