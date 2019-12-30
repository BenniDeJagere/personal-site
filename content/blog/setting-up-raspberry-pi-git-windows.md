+++
date = "2016-03-02T14:40:08+01:00"
draft = false
title = "Setting up a Raspberry Pi as a git server for Windows clients"
tags = ["git", "Linux", "Windows"]
+++

Sometimes you're not allowed or you don't want to publish source code on GitHub, Bitbucket etc. In this post, I'll explain how you can set up a Raspberry Pi (or any other Linux server) as a git repository server and how you can configure Windows clients for that server. We'll automate everything from authentication to creating and deleting repositories.

## Installation of the prerequisites

### On the Windows clients

On your Windows clients, you'll need PuTTY, Pageant, Plink, PuTTYgen and Git Bash. The installers are very straightforward.

First install [PuTTY and its related tools](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html). After the installation, add the installation directory to your `PATH`.

```
Windows 10:
System properties > Advanced system settings > Environment variables > System variables > Path > Add new >
C:\Program Files (x86)\PuTTY

Windows 8.1 or earlier: the directories in the Path variable are separated with a ;
```

When that's done, install [Git and its utilities](https://git-for-windows.github.io/). During the installation, select the following options:

* Use Git and optional Unix tools from the Windows Command Prompt
* Use (Tortoise)Plink: `C:\Program Files (x86)\PuTTY\plink.exe`
* Checkout Windows-style, commit Unix-style line endings
* Use Windows' default console window

### On the Raspberry Pi / Linux server

You'll need ssh and git. On Debian-based systems, you can install them with the command below:

```shell
sudo apt-get update && apt-get install -y openssh-server git
```

I'd recommend to [setup SSH](http://www.cyberciti.biz/faq/debian-linux-install-openssh-sshd-server/) and PuTTY for your personal user account and use that to perform the tasks in this guide.

## Setting up the Linux server

If you're using a Raspberry Pi, I'd suggest not using your SD card to store your git repositories. Instead, use an external hard drive or a USB drive. I'm not going to explain how to mount a USB drive, as there are already [a lot of guides](http://www.raspberrypi-spy.co.uk/2014/05/how-to-mount-a-usb-flash-disk-on-the-raspberry-pi/) explaining this.

First, we'll have to create a new user and use its home directory as the root of our git repositories.

```shell
sudo adduser --home /path/to/where/you/want/to/store/your/repos --disabled-password gituser
```

The command will ask you a few details about the user which you can safely ignore by pressing *Enter*.

Next up, we'll be configuring the SSH daemon. Use `sudo nano /etc/ssh/sshd_config` to open up a text editor with the configuration file. Make sure it contains the lines below, so paste them in (*right click*) or modify the existing configuration options.

```
PubkeyAuthentication yes # enable key-based authentication
AuthorizedKeysFile %h/.ssh/authorized_keys # where the authorized keys are stored
PermitRootLogin no # security measure: do not allow root to sign in using ssh
AllowUsers gituser # make sure our user is allowed to connect using SSH, add other users here separated with spaces if you want to allow them to use SSH
```

Save with *CTRL+S* and exit with *CTRL+X*.

When that's done, it's time to log in as the newly created user with `sudo su gituser` and go to its home directory using `cd`.

Create a file to store the SSH authentication key with the right permissions:

```shell
mkdir .ssh
chdmod 700 .ssh
cd .ssh
touch authorized_keys
chmod 600 authorized_keys
nano authorized_keys
```

This will open up a text editor.

Now, back to our Windows machine. Let's generate a new key using PuTTYgen. I usually create a SSH-2 RSA key consisting of 4096 bits. Save the private key somewhere on your machine and copy the generated public key and paste it in the text editor on the Linux machine.

On the Linux machine, save the file and exit the text editor. To make sure the new configuration is active, restart the SSH server using `sudo service ssh restart`.

Because we want to automate the creation and deletion of git repositories, we'll need two scripts. I've provided them below.

{{< gist sdebruyn 5c7004e92b9c293acc5a >}}

Use the following steps to put them on your system and make them runnable:

1. `cd`
1. `nano delrepo.sh`
1. Paste script 1 using *right click*
1. Save and exit the text editor (*CTRL+S*, *CTLRL+X*)
1. `nano initrepo.sh`
1. Paste script 2 using *right click*
1. Save and exit the text editor (*CTRL+S*, *CTLRL+X*)
1. `chmod +x ./*repo.sh`

This concludes the setup on our Linux box.

## Configuring the Windows clients

Time to fire up Pageant and add the private key we created before. Afterwards, fire up PuTTY and configure it to connect to the Linux machine.

You need to supply the following configuration options:

* Session > Host name / IP address: address of the Linux machine
* Session > Port: default is 22 (can be set in *sshd_config* with `Port 12345`)
* Connection > Data > Auto-login username: username of the user you created (I used *gituser*)
* Connection > SSH > Auth > Authentication methods > Attempt authentication using Pageant: must be checked
* Connection > SSH > Auth > Authentication methods > Attempt "keyboard-interactive" auth: should not be checked

Make sure to save the session (Session > Saved sessions - I saved it as *gituser*). Then open the connection. PuTTY will warn you about the new host, so type `yes` and press *Enter* to confirm. You should now be successfully connected to the Linux box using SSH. You can close the session if that's the case, otherwise verify if you've followed every step in this guide correctly and if you can ping the Linux machine.

To complete our setup, we need to make Plink the SSH agent for git. All you need to do is create a new environment variable called `GIT_SSH`.

```
System properties > Advanced system settings > Environment variables > System variables > New >
Name: GIT_SSH
Value: C:\Program Files (x86)\PuTTY\plink.exe
```

To create a new repository to use as a remote, open a Windows Command Prompt and type `plink name-of-the-saved-session ./initrepo.sh name-of-the-new-repo`. To delete an existing repository, you can use `plink name-of-the-saved-session ./delrepo.sh name-of-the-existing-repo`.

To add the repository you created as a remote to a git repository, use `git remote add how-you-want-to-name-the-remote ssh://address-of-the-linux-machine/~/name-of-the-repository.git`. If you're not using port 22 for SSH, put the port after the address, separated with colon.

Because we use Pageant for authentication, we need to load Pageant with the private key every time we boot up our Windows machine. This is the last part of the process we have to automate. Create a new shortcut to Pageant in the folder `%appdata%\Microsoft\Windows\Start Menu\Programs\Startup`. Now add the keys to that shortcut as explained in [this guide](http://blog.shvetsov.com/2010/03/making-pageant-automatically-load-keys.html).

So this was my guide to setting up a Linux machine as a git repository server for Windows clients. You can now start committing and pushing away!