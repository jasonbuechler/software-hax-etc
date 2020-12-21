## Basic config for a new FreeBSD/FreeNAS jail
### enable SSH access so we can use Putty/etc
---
```
passwd
pkg install nano
grep -ni root /etc/ssh/sshd_config
  # 37:#PermitRootLogin no
  # 83:# the setting of "PermitRootLogin without-password".
  # 107:#ChrootDirectory none
nano /etc/ssh/sshd_config
service sshd start
sysrc sshd_enable="YES"
service sshd start
service sshd status
```

# Find pkg-installable releases of req'd software
## semi-optional demonostrative exercise
```
pkg search -g "mediawiki*"
  # mediawiki131-php72-1.31.10     Wiki engine used by Wikipedia
  # mediawiki131-php73-1.31.10     Wiki engine used by Wikipedia
  # mediawiki131-php74-1.31.10     Wiki engine used by Wikipedia
  # mediawiki133-php72-1.33.3      Wiki engine used by Wikipedia
  # mediawiki133-php73-1.33.3      Wiki engine used by Wikipedia
  # mediawiki133-php74-1.33.3      Wiki engine used by Wikipedia
  # mediawiki134-php72-1.34.4      Wiki engine used by Wikipedia
  # mediawiki134-php73-1.34.4      Wiki engine used by Wikipedia
  # mediawiki134-php74-1.34.4      Wiki engine used by Wikipedia
  # mediawiki135-php72-1.35.0      Wiki engine used by Wikipedia
  # mediawiki135-php73-1.35.0      Wiki engine used by Wikipedia
  # mediawiki135-php74-1.35.0      Wiki engine used by Wikipedia

pkg search -d mediawiki135-php74 | grep -v "php74-[a-z]"
  # mediawiki135-php74-1.35.0
  # Comment        : Wiki engine used by Wikipedia
  # Depends on     :
  #     php74-7.4.13
  #     mysql57-client-5.7.32


pkg search -g "mysql57*"
  # mysql57-client-5.7.32 |  Multithreaded SQL database (client)
  # mysql57-server-5.7.32 |  Multithreaded SQL database (server)

pkg search -g "apache*" | grep server
  # apache24-2.4.46       | Version 2.4.x of Apache web server

pkg search -g "mod_php*"
  # mod_php72-7.2.34      |  PHP Scripting Language
  # mod_php73-7.3.25      |  PHP Scripting Language
  # mod_php74-7.4.13      |  PHP Scripting Language
```

# Install all 4 packages simultaneously
## match-up versions for php and mysql
```
pkg install mediawiki135-php74 \
            apache24 \
            mysql57-server \
            mod_php74
pkg info
```

# Review developer install notes for useful tips
## same messages that print after install, above
```
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

# Collect some info we'll use later
## semi-optional demonstrative exercise
```
ls -a ~/
  # .               .cshrc          .lesshst        .profile
  # ..              .k5login        .login
pkg list mediawiki135-php74 | grep mediawiki/index.php
  # /usr/local/www/mediawiki/index.php
pkg list apache24 | grep httpd.conf
  # /usr/local/etc/apache24/httpd.conf.sample
```

# Start web & database services
## following up on the useful apache note
```
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

# Collect some more info
## semi-optional demonstrative exercise
```
ls -a /root
  # .               .cshrc          .lesshst        .mysql_secret
  # ..              .k5login        .login          .profile
ls /usr/local/etc/apache24/
  # Includes                httpd.conf.sample       mime.types.sample
  # envvars.d               magic                   modules.d
  # extra                   magic.sample
  # httpd.conf              mime.types
ls /usr/local/etc/apache24/modules.d/
  # README_modules.d
cat /usr/local/etc/apache24/modules.d/README_modules.d
  # Directory for third party module config files.
  # Files are automatically included if the name
  # begins with a three digit number followed by '_'
  # and ending in '.conf' e.g. '080_mod_php.conf'
grep -C1 index.html /usr/local/etc/apache24/httpd.conf
  # <IfModule dir_module>
  #     DirectoryIndex index.html
  # </IfModule>
grep -ni documentroot /usr/local/etc/apache24/httpd.conf
  # 247:# DocumentRoot: The directory out of which you will serve your
  # 251:DocumentRoot "/usr/local/www/apache24/data"
  # 351:# access content that does not live under the DocumentRoot.
```

# Take some shortcuts & setup config files
## following up on that apache readme
```
pkg list mediawiki135-php74 | grep mediawiki/index.php \
  >   /usr/local/etc/apache24/modules.d/099_mediawiki.conf
cat /usr/local/etc/apache24/httpd.conf | \
  grep -A99 -i ^documentroot | grep -B99 -m1 "</Directory>" \
  >>  /usr/local/etc/apache24/modules.d/099_mediawiki.conf

pkg info -D mod_php74 | grep -i -e filesmatch -e sethandler \
  >   /usr/local/etc/apache24/modules.d/080_mod_php.conf
cat /usr/local/etc/apache24/httpd.conf | grep -C1 index.html \
  >>  /usr/local/etc/apache24/modules.d/080_mod_php.conf

nano /usr/local/etc/apache24/httpd.conf
nano /usr/local/etc/apache24/modules.d/080_mod_php.conf
nano /usr/local/etc/apache24/modules.d/099_mediawiki.conf
```

# Check configs and tell apache to reload them
## Technically, reload alone should do both
```
service apache24 configtest
service apache24 reload
```

---

# Notes

re: general procedure
https://www.digitalocean.com/community/tutorials/how-to-install-an-apache-mysql-and-php-famp-stack-on-freebsd-12-0

re: mod_php necessaries
https://websistent.com/change-the-default-directory-index/
