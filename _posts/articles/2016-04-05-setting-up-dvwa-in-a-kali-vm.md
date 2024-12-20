---
layout: post
title: "Setting Up DVWA in a Kali VM"
modified:
categories: articles
excerpt: "A guide to installing a vulnerable Web application in a security-testing oriented Linux distribution."
tags: [software, security, php, Linux]
date: 2016-04-05
---

## Why DVWA?

[Damn Vulnerable Web App](http://www.dvwa.co.uk/) is exactly what it sounds like.  It's a useful way to learn about some of the most common security problems that Web application developers face.  While DVWA uses some pretty unhip technologies (PHP5 and MySQL), most of the vulnerabilities apply to all sorts of Web apps.

The creators of DVWA recommend that you play with it in a VM sandbox.  This can be done in a few ways:

1. Use [Vagrant](https://www.vagrantup.com/) and [Scotch-Box](https://box.scotch.io/) (see TL;DD at the end of this post)
2. Install XAMPP into a vanilla VM
3. Install from the [DVWA LiveCD](http://www.dvwa.co.uk/DVWA-1.0.7.iso) and then update DVWA itself to the newest version (1.9 as of this writing)
4. Install DVWA in a Kali Linux VM

I'll be outlining solution #4.

## Why Kali?
[Kali Linux](https://www.kali.org/) is a Debian-based distribution that comes with security and penetration testing tools like [Burp Suite](https://portswigger.net/burp/) and [Metasploit](https://www.metasploit.com/).  This may be overkill for DVWA, but anyone looking at DVWA is probably interested in learning more about security anyway.

Another reason is that the kind people at [Offensive Security](https://www.offensive-security.com/) have created [pre-made VirtualBox and VMWare images of Kali](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/), which saves us some time.

## The process

### What you'll need
1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) installed
2. The prebuilt [VirtualBox image of Kali](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/).  I recommend downloading it via BitTorrent.
3. [DVWA itself](http://www.dvwa.co.uk/) (use the download link there for the latest version)
4. Something that can decompress .7z files ([7-Zip](http://www.7-zip.org/) on Windows, [The Unarchiver](https://itunes.apple.com/us/app/the-unarchiver/id425424353) on a Mac, or p7zip in Linux)

### Kali Setup
1. Decompress the .7z with the VirtualBox image.  This should yield a `.ova` file.
2. In VirtualBox Manager, go to `File -> Import Appliance...` and select the appropriate .ova file.  You can just click `Continue` and `Import`.  The default settings should be fine.  This part may take a few minutes.
3. Start the Kali VM and login as root.  The default password for the VM is `toor`.
4. Open up a terminal window from the icon with a "$_" on the left of the screen.
5. [Create a new user](https://www.linkedin.com/pulse/20140502074357-79939846-adding-a-new-user-in-kali-linux).
    - `useradd -m <username>`
    - `passwd <username>` to set a password
    - `usermod -a -G sudo <username>` to add the user to the sudo group
    - `chsh -s /bin/bash <username>` to set the shell to bash (or another shell if you prefer)
6. Logout as root, then login as the new user.

### DVWA Installation
1. Drag the `DVWA-1.9.zip` file onto your VM's desktop (or just open IceWeasel and download it).
2. Open a terminal window and run the following:
    - `sudo su`
    - `unzip /home/<your username from above>/Desktop/DVWA-1.9.zip -d /var/www/html/`
    - `cd /var/www/html`
    - `mv DVWA-1.9/ dvwa/`
    - `chmod -R 755 dvwa`
    - `service mysql start`
    - `mysql -u root -p`. This will ask you for a password which should be blank.
    - `create database dvwa;`
    - `exit` to get back to the shell
    - `nano dvwa/config/config.inc.php` and change the db_password to `''` (empty). 
    - `service apache2 start`

You should now be able to open the browser (IceWeasel) and navigate to [http://localhost/dvwa/setup.php](http://localhost/dvwa/setup.php).

### Fixing some things
You'll see that a few items on the setup checklist are red.  This will keep us from being able to do all of the exercises included with DVWA.

1. At the bottom, we see that a couple of folders need to be made writable by the web server.  Let's do:
    - `chgrp www-data dvwa/hackable/uploads/`
    - `chmod g+w dvwa/hackable/uploads/`
    - `chgrp www-data dvwa/external/phpids/0.6/lib/IDS/tmp/phpids_log.txt`
    - `chmod g+w dvwa/external/phpids/0.6/lib/IDS/tmp/phpids_log.txt`
2. Install the php-gd image manipulation module for PHP5:
    - a simple `apt-get install php5-gd` won't work here.  We'll need to add a repository first:
    - while still in `su` mode, `nano /etc/apt/sources.list`
    - at the very end of the file, add the line `deb http://repo.kali.org/kali kali-rolling main non-free contrib` and save the file
    - `apt-get update`
    - `apt-get install php5-gd`
3. Enable `allow_url_include`:
    - `nano /etc/php5/apache2/php.ini` 
    - change the `allow_url_include` from `Off` to `On`
4. You can [create a reCAPCHA key](https://www.google.com/recaptcha/admin/create) and add it to config.inc.php if you want.
5. `service apache2 restart` to make the configuration changes live.

Now reload the setup.php page.  Everything should be green and we can hit the "Create / Reset Database" button at the bottom.

## If you want to work from outside the VM:
Everything should be working now, but we may prefer to use our host OS to do our actual work with DVWA.  We'll need to change a network setting and install an SSH server.

### Switch to Bridged Adapter mode
VirtualBox probably defaulted the guest OS's network to NAT mode.  This worked fine for our apt-gets and such, but won't let us access the Apache web server.  In our VM's machine -> preferences menu, go to the Network tab and set `Attached to:` to `Bridged Adapter`.  It may take a moment for the changes to take place.  See the [VirtualBox manual](https://www.virtualbox.org/manual/ch06.html) for more information on the different networking modes.

### Setup SSH
You can follow [this guide by a guy who calls himself Dr.Chaos](http://www.drchaos.com/enable-ssh-on-kali-linux-enable-ssh-on-kali-linux/) to get SSH working (you'll need to run the commands as superuser).  You can skip steps 4 and 5.  Since we already made a user in the superuser group, we can just log in as that user instead of as root.

### Access to DVWA on your Host OS's browser
In the VM's terminal, type `hostname -I` to get the IP address.  Point your browser to `http://<kali-ip>/dvwa/`.

### SSH into into Kali
If MacOS is your host OS, you can type `ssh <username-we-created>@<kali-ip>` in the terminal to connect.

### Setup a network share
Follow [this guide on installing samba](https://help.ubuntu.com/community/How%20to%20Create%20a%20Network%20Share%20Via%20Samba%20Via%20CLI%20(Command-line%20interface/Linux%20Terminal)%20-%20Uncomplicated,%20Simple%20and%20Brief%20Way!).  

I added the following section to my `/etc/samba/smb.conf`:

```
[dvwa]
   comment = DVWA Folder
   path = /var/www/html/dvwa
   writeable = yes
   browseable = yes
   public = yes
   read only = no
   create mask = 0644
   force create mode = 0755
   valid users = <user-we-created>
```

Add the user we created in this guide when the guide above tells you to add a Samba user.

To be able to actually change the files through the Samba share, you'll need to `sudo chown -R <user-we-created> /var/www/html/dvwa`

Now connect to `smb://<user-we-created>@<kali-ip>/dvwa` in Finder/Explorer/Nautilus/etc.

You should be able to point your favorite editor to the share and make changes to the files.  In OSX, it should be in the folder `/Volumes/dvwa`.  If your preferred editor is Sublime Text, you could run `subl /Volumes/dvwa`.


## All Set!

Now you can navigate to the login page at [http://localhost(or-kali-ip)/dvwa/login.php](http://localhost/dvwa/login.php).  The default credentials are admin/password.

You can shut down the VM and save state to pick up where you left off.

**NOTE:** If you see a launch-bar in the VM but it otherwise seems unresponsive, try pressing Enter.  The screen has auto-locked and you'll have to re-enter your password.

Thanks to [Linh Nguyen](https://theotherlinh.com/) for some useful notes and input.

## TL;DD:
If you don't care about any of the tools that come with Kali, you can use Vagrant and Scotch-Box instead.  Start out by following the [get started guide on the Scotch-Box site](https://box.scotch.io/#get-started).

* You will still need to edit DVWA's `config/config.inc.php` to change the MySQL password to `root`.  
* You'll also need to enable `allow_url_include` in the VM's `etc/php5/apache2/php.ini`
    * From within your project's folder: `vagrant ssh`
    * `sudo nano /etc/php5/apache2/php.ini`
    * Ctrl+W, `allow_url_include`, change it to `On`
    * Ctrl+X, then say [Y]es.
    * `sudo service apache2 restart`
    * `exit`