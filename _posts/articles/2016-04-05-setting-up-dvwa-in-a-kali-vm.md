---
layout: post
title: "Setting Up DVWA in a Kali VM"
modified:
categories: articles
excerpt: "A guide to installing a vulnerable Web application in a security-testing oriented Linux distribution."
tags: [software, security, php, Linux]
image:
  feature:
date: 2016-04-05
---

## Why DVWA?

[Damn Vulnerable Web App](http://www.dvwa.co.uk/) is exactly what it sounds like.  It's a useful way to learn about some of the most common security problems that Web application developers face.  While DVWA uses some pretty unhip technologies (PHP5 and MySQL), most of the vulnerabilities apply to all sorts of Web apps.

The creators of DVWA recommend that you play with it in a VM sandbox.  This can be done in a few ways:

1. Use [Vagrant](https://www.vagrantup.com/) and [Scotch-Box](https://box.scotch.io/)
2. Install XAMPP into a vanilla VM
3. Install from the [DVWA LiveCD](http://www.dvwa.co.uk/DVWA-1.0.7.iso) and then update DVWA itself to the newest version (1.9 as of this writing)
4. Install DVWA in a Kali Linux VM

Our solution is #4.

## Why Kali?
[Kali Linux](https://www.kali.org/) is a Debian-based distribution that comes with security and penetration testing tools like [Burp Suite](https://portswigger.net/burp/) and [Metasploit](https://www.metasploit.com/).  This may be overkill for DVWA, but anyone looking at DVWA is probably interested in learning more about security anyway.

Another reason is that the kind people at [Offensive Security](https://www.offensive-security.com/) have created [pre-made VirtualBox and VMWare images of Kali](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/), which saves us some time.

## The process

### What you'll need
1. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 
2. The prebuilt [VirtualBox image of Kali](https://www.offensive-security.com/kali-linux-vmware-virtualbox-image-download/)
3. [DVWA itself](https://github.com/RandomStorm/DVWA/archive/v1.9.zip)
4. Something that can decompress .7z files

If your Kali download fails, try using the torrent.  Extract the Kali image archive before proceeding.

### Kali Setup
1. In VirtualBox Manager, go to `File -> Import Appliance...` and select the appropriate .ova file.  You can just click `Continue` and `Import`.  The default settings should be fine.
2. Start the Kali VM and login as root.  The default password for the VM is `toor`.
3. [Create a new user](https://www.linkedin.com/pulse/20140502074357-79939846-adding-a-new-user-in-kali-linux).
    - `useradd -m username`
    - `passwd username` to set a password
    - `usermod -a -G sudo username` to add the user to the sudo group
    - `chsh -s /bin/bash username` to set the shell to bash (or another shell if you prefer)
4. Login as the new user

### DVWA Installation
1. Drag the `DVWA-1.9.zip` file onto your VM's desktop (or just open IceWeasel and download it).
2. Open a terminal window and run the following:
    - `sudo su`
    - `unzip ~/Desktop/DVWA-1.9.zip -d /var/www/html/`
    - `cd /var/www/html`
    - `mv DVWA-1.9/ dvwa/`
    - `chmod -R 755 dvwa`
    - `service mysql start`
    - `mysql -u root -p`. This will ask you for a password which should be blank.
    - `create database dvwa;`
    - `exit` to get back to the shell
    - `nano dvwa/config/config.inc.php` and change the db_password to `''` (empty)
    - `service apache2 start`

You should now be able to open the browser (IceWeasel) and navigate to [the DVWA directory on localhost](http://localhost/dvwa/).

### Fixing some things
DVWA will send us to setup.php to take care of a few things before we can login.  You'll see that a few items on the checklist are red.

1. At the bottom, we see a couple of folders need to be made writable by the web server.  Let's do:
    - `chgrp www-data dvwa/hackable/uploads/`
    - `chmod g+w dvwa/hackable/uploads/`
    - `chgrp www-data dvwa/external/phpids/0.6/lib/IDS/tmp/phpids_log.txt`
    - `chmod g+w dvwa/external/phpids/0.6/lib/IDS/tmp/phpids_log.txt`
2. Enable `allow_url_include`:
    - `nano /etc/php5/apache2/php.ini` 
    - change the `allow_url_include` from `Off` to `On`
    - `service apache2 restart`
3. Install the php-gd image manipulation module
    - a simple `apt-get install php-gd` won't work here.  We'll need to add a repository first:
    - while still in `su` mode, `nano /etc/apt/sources.list`
    - at the very end of the file, add the line `deb http://repo.kali.org/kali kali-rolling main non-free contrib` and save the file
    - `apt-get update`
    - `apt-get install php5-gd`

4. You can [create a reCAPCHA key](https://www.google.com/recaptcha/admin/create) and add it to config.inc.php if you want.

Now do one last `service apache2 restart` and reload the setup.php page.  Everything should be green and we can hit the "Create / Reset Database" button at the bottom.

## All Set!

You should now be redirected to the [login page](http://localhost/dvwa/login.php).  The default credentials are admin/password.

You can shut down the VM and save state to pick up where you left off.