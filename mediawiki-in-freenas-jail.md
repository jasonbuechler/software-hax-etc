# Demystifying mediawiki installation 
#### Installing and configuring an *\*AMP* stack "by hand" isn't complex if you are willing to give it a try

This document specifically uses a FreeNAS12-based jail install... but outside of idiosyncrasies that exist in different \*nix systems, the following directions are pretty generic.  

By "idiosyncrasies", I mean whatever isn't identical to a new, stock FreeNAS jail. Among other things, this probably means your command-line interface shell (`csh` vs `bash` vs ...), your package manager (`pkg` vs `apt` vs `yum` vs ...), and the individual packages available to your system's package manager.  (One very notable example is that mediawiki+php come as one convenient bundle in the pkg ecosystem... so if you're working with redhat, for example, you might have to download/wget/git-clone mediawiki to your system and check its requirements for the necessary PHP version, which you would then install separately through your package manager.)

In the first section below, I'm assuming you've just used the FreeNAS jail manager to create a new jail (with automatic DHCP) and then used the jail-manager to open a console-shell into the brand new jail.  Since that console can be annoying to use (control-w and copy/paste in particular) my first step is enabling the SSH service and allowing 'root' to connect.

---
### Basic config for a new FreeBSD/FreeNAS jail
#### *enable SSH access so we can use Putty/etc*
Use nano to change `#PermitRootLogin no` to `PermitRootLogin yes`.
```bash
passwd
pkg install nano
grep -ni root /etc/ssh/sshd_config
  # 37:#PermitRootLogin no
  # 83:# the setting of "PermitRootLogin without-password".
  # 107:#ChrootDirectory none
nano /etc/ssh/sshd_config
service sshd start
  # Cannot 'start' sshd. Set sshd_enable to YES in
  # /etc/rc.conf or use 'onestart' instead of 'start'.
sysrc sshd_enable="YES"
service sshd start
service sshd status
echo "Now SSH to <IP of jail> using Putty/Terminal/etc!"
```

---
### Find pkg-installable releases of req'd software
#### *semi-optional demonostrative exercise*
Searching packages can vary a bit between package managers: here I'm using the `-g` flag to enable shell-style "globbing" (using an asterisk to fill in for "anything else"), and the `-d` flag to list the direct dependencies of a given package.
```bash
pkg search -g "mediawiki*"
  # mediawiki133-php72-1.33.3     Wiki engine used by Wikipedia
  # mediawiki133-php73-1.33.3     Wiki engine used by Wikipedia
  # mediawiki133-php74-1.33.3     Wiki engine used by Wikipedia
  # mediawiki134-php72-1.34.4     Wiki engine used by Wikipedia
  # mediawiki134-php73-1.34.4     Wiki engine used by Wikipedia
  # mediawiki134-php74-1.34.4     Wiki engine used by Wikipedia
  # mediawiki135-php72-1.35.0     Wiki engine used by Wikipedia
  # mediawiki135-php73-1.35.0     Wiki engine used by Wikipedia
  # mediawiki135-php74-1.35.0     Wiki engine used by Wikipedia

pkg search -d mediawiki135-php74 | grep -v "php74-[a-z]"
  # mediawiki135-php74-1.35.0
  # Comment        : Wiki engine used by Wikipedia
  # Depends on     :
  #     php74-7.4.13
  #     mysql57-client-5.7.32

pkg search -g "mysql57*"
  # mysql57-client-5.7.32    Multithreaded SQL database (client)
  # mysql57-server-5.7.32    Multithreaded SQL database (server)

pkg search -g "apache*" | grep server
  # apache24-2.4.46         Version 2.4.x of Apache web server

pkg search -g "mod_php*"
  # mod_php72-7.2.34         PHP Scripting Language
  # mod_php73-7.3.25         PHP Scripting Language
  # mod_php74-7.4.13         PHP Scripting Language
```

---
### Install all 4 packages simultaneously
#### *match-up versions for php and mysql*
```bash
pkg install mediawiki135-php74 \
            apache24 \
            mysql57-server \
            mod_php74
pkg info
```

---
### Review developer install notes for useful tips
#### *(The info that gets printed after installs)*
The `pkg` package manager can print these messages at-will using the `info -D` subcommand. We will be putting several of the items to use, later.
```bash
pkg info -D apache24
  # To run apache www server from startup, add apache24_enable="yes"
  # in your /etc/rc.conf. Extra options can be found in startup script.

  # Please compare the existing httpd.conf with httpd.conf.sample
  # and merge missing modules/instructions into httpd.conf!

pkg info -D mod_php74
  # Make sure index.php is part of your DirectoryIndex.
  # You should add the following to your Apache configuration file:
  # <FilesMatch "\.php$">
  #     SetHandler application/x-httpd-php
  # </FilesMatch>
  # <FilesMatch "\.phps$">
  #     SetHandler application/x-httpd-php-source
  # </FilesMatch>

pkg info -D mysql57-server
  # Initial password for first time use of MySQL is saved in 
  # $HOME/.mysql_secret
  # ie. when you want to use "mysql -u root -p" first you should see 
  # password in /root/.mysql_secret
```

---
### Collect some info we'll use later
#### *semi-optional demonstrative exercise*
The `list` subcommand prints the name & path of each file installed. We will be using a couple key files to know where and how to configure things. We'll use Apache's main config directory a lot later... so I'm also establishing a variable with the path, as a shortcut.
```bash
ls -a ~/
  # .               .cshrc          .lesshst        .profile
  # ..              .k5login        .login
  
pkg list mediawiki135-php74 | grep mediawiki/index.php
  # /usr/local/www/mediawiki/index.php
pkg list apache24 | grep httpd.conf
  # /usr/local/etc/apache24/httpd.conf.sample
  
set apacheDir=/usr/local/etc/apache24
echo "apache configs live in $apacheDir"
  # apache configs live in /usr/local/etc/apache24
```

---
### Start web & database services
#### *following up on the useful apache note*
Apache's install note mentioned how to enable the service, but you can often get the same info by trying a pre-emptive `service XYZ start` command.
```bash
service apache24 start
  # Cannot 'start' apache24. Set apache24_enable to YES 
  # in /etc/rc.conf or use 'onestart' instead of 'start'.
sysrc apache24_enable="yes"
service apache24 start
service apache24 status

service mysql-server start
  # Cannot 'start' mysql. Set mysql_enable to YES in 
  # /etc/rc.conf or use 'onestart'instead of 'start'.
sysrc mysql_enable=YES
service mysql-server start
service mysql-server status
```

---
### Collect some more info
#### *semi-optional demonstrative exercise*
```bash
echo $apacheDir
  # /usr/local/etc/apache24
ls -a /root
  # .               .cshrc          .lesshst        .mysql_secret
  # ..              .k5login        .login          .profile
cat /root/.mysql_secret
  # <random passcode>  ## NOTE THIS FOR LATER
ls $apacheDir
  # Includes                httpd.conf.sample       mime.types.sample
  # envvars.d               magic                   modules.d
  # extra                   magic.sample
  # httpd.conf              mime.types
ls $apacheDir/modules.d/
  # README_modules.d
cat $apacheDir/modules.d/README_modules.d
  # Directory for third party module config files.
  # Files are automatically included if the name
  # begins with a three digit number followed by '_'
  # and ending in '.conf' e.g. '080_mod_php.conf'
cat $apacheDir/httpd.conf | grep -C1 index.html
  # <IfModule dir_module>
  #     DirectoryIndex index.html
  # </IfModule>
cat $apacheDir/httpd.conf | grep -ni documentroot
  # 247:# DocumentRoot: The directory out of which you will serve your
  # 251:DocumentRoot "/usr/local/www/apache24/data"
  # 351:# access content that does not live under the DocumentRoot.
cat $apacheDir/httpd.conf | grep -i "^documentroot"
  # DocumentRoot "/usr/local/www/apache24/data"
```

---
### Take some shortcuts & setup config files
#### *following up on that apache readme*
Use nano to comment-out the DocumentRoot line of the main conf...<br>
Use nano to add index.php after index.html in the php conf...<br>
Use nano to set DirectoryRoot + directives in the mediawiki conf. 
```bash
pkg list mediawiki135-php74 | grep "mediawiki/index.php" \
  >   $apacheDir/modules.d/099_mediawiki.conf
cat /usr/local/etc/apache24/httpd.conf | \
  grep -A99 -i ^documentroot | grep -B99 -m1 "</Directory>" \
  >>  $apacheDir/modules.d/099_mediawiki.conf

pkg info -D mod_php74 | grep -i -e filesmatch -e sethandler \
  >   $apacheDir/modules.d/080_mod_php.conf
cat $apacheDir/httpd.conf | grep -C1 "index.html" \
  >>  $apacheDir/modules.d/080_mod_php.conf

nano $apacheDir/httpd.conf
nano $apacheDir/modules.d/080_mod_php.conf
nano $apacheDir/modules.d/099_mediawiki.conf
```

---
### Check configs and tell apache to reload them
#### *Technically, reload alone should do both*
```bash
service apache24 configtest
service apache24 reload
service apache24 status
echo "done!"
```

---
### Finish MySQL setup by setting new password
#### *Otherwise database access is restricted*
Be very, very careful with your spacing and quote-punctution. No space can be used when passing the login-password, even if you use the short form "-p" instead. I think the punctuation below should work for any shell, if used exactly.
The awk command pulls the 2nd line out of the pre-expired password file. (and the 'oldPw' variable is set to that value.)
```bash
set oldPw=`awk NR==2 /root/.mysql_secret`
mysqladmin --user=root --password="$oldPw" password 'newPassword'
mysqladmin --user=root --password="newPassword" status
echo "if no errors above, then you're done!"
```

---
### Complete the mediawiki "wizard" 
#### http://\<IP of jail\>/ (or http://\<IP\>/mw-config/index.php)
For the purposes of this document, just use 'root' for the mysql user, and whatever password you used instead of "newPassword", above.
You'll obviously want to update everything to be more secure later. And similarly, also run `mysql_secure_installation`.

---
### Notes

re: general procedure<br>
https://www.digitalocean.com/community/tutorials/how-to-install-an-apache-mysql-and-php-famp-stack-on-freebsd-12-0


re: mod_php necessaries<br>
https://websistent.com/change-the-default-directory-index/
