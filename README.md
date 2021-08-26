# GLPI on Manjaro

<br>

The objective of this documentation is to provide a step by step about how to install GLPI on Manjaro.

<br>

For this presentation, consider the following scenario:
* Linux 5.10.59-1-MANJARO
* Apache/2.4.48 (Unix)
* PHP 8.0.9 (cli) (built: Jul 31 2021 08:10:26) ( NTS )
* MariaDB 10.6.4 Arch Linux
* GLPI 9.5.5

<br>

So... Let's go!

<br>
<hr>
<br>

## LAMP

<br>

> **LAMP** stands for *Linux*, *Apache*, *MySQL*, and *PHP*. Together, they provide a proven set of software for delivering high-performance web applications.
> - [IBM Cloud Education](https://www.ibm.com/cloud/learn/lamp-stack-explained)

<br>

As [GLPI prerequisite](https://glpi-install.readthedocs.io/en/latest/prerequisites.html), we need to have the LAMP stack up and running.

<br>

### Linux

**Step 1** - Update the system repository using pacman.

    sudo pacman -Syyu

<br>
<hr>
<br>

### Apache

**Step 1** - Installation

    sudo pacman -S apache

<br>

**Step 2** - Configure the httpd.conf file (/etc/httpd/conf/httpd.conf).

    sudo nano /etc/httpd/conf/httpd.conf

Comment the `LoadModule unique_id_module modules/mod_unique_id.so` line

    # LoadModule unique_id_module modules/mod_unique_id.so

<br>

**Step 3** - Enable Apache on system startup

    sudo systemctl enable httpd

<br>

**Step 4** - Restart Apache service

    sudo systemctl restart httpd

<br>

**Step 5** - Verify Apache status

    sudo systemctl status httpd

The result of this command will show a lot of information. What we need to focus right now is on the `active (running)` status field.

![httpd.service - Apache Web Server](/images/manjaro-glpi-apache-status.png)

<br>
<hr>
<br>

### MySQL

**Step 1** - Installation

    sudo pacman -S mysql

If asked during the installation...

    :: There are 2 providers available for mysql:
    :: Repository extra
    1) mariadb
    :: Repository community
    2) percona-server

Choose option 1, mariadb.

<br>

**Step 2** - Initialize MariaDB data directory

    sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

<br>

**Step 3** - Enable MariaDB on system startup

    sudo systemctl enable mysqld

<br>

**Step 4** - Start MariaDB service

    sudo systemctl start mysqld

<br>

**Step 5** - Verify MariaDB status

    sudo systemctl status mysqld

The result of this command will show a lot of information. What we need to focus right now is on the `active (running)` status field.

![mariadb.service - MariaDB 10.6.4 database server](/images/manjaro-glpi-mariadb-status.png)

<br>

**Step 6** - Run the `mysql_secure_installation` script

    sudo mysql_secure_installation

The script will ask some questions:

    NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
    SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

    In order to log into MariaDB to secure it, we'll need the current
    password for the root user. If you've just installed MariaDB, and
    haven't set the root password yet, you should just press enter here.

    Enter current password for root (enter for none):

Press `Enter`;

    OK, successfully used password, moving on...

    Setting the root password or using the unix_socket ensures that nobody
    can log into the MariaDB root user without the proper authorisation.

    You already have your root account protected, so you can safely answer 'n'.

    Switch to unix_socket authentication [Y/n]

Press `n`, then *Enter*.

    ... skipping.

    You already have your root account protected, so you can safely answer 'n'.

    Change the root password? [Y/n]

Press `Y`, then *Enter*.

    New password:
    Re-enter new password:

Type your password and press *Enter*. Then, type your password and press *Enter* again, as confirmation;

    Password updated successfully!
    Reloading privilege tables..
    ... Success!


    By default, a MariaDB installation has an anonymous user, allowing anyone
    to log into MariaDB without having to have a user account created for
    them.  This is intended only for testing, and to make the installation
    go a bit smoother.  You should remove them before moving into a
    production environment.

    Remove anonymous users? [Y/n]

Press `Y`, then *Enter*.

    ... Success!

    Normally, root should only be allowed to connect from 'localhost'.  This
    ensures that someone cannot guess at the root password from the network.

    Disallow root login remotely? [Y/n]

Press `Y`, then *Enter*.

    ... Success!

    By default, MariaDB comes with a database named 'test' that anyone can
    access.  This is also intended only for testing, and should be removed
    before moving into a production environment.

    Remove test database and access to it? [Y/n]

Press `Y`, then *Enter*.

    - Dropping test database...
    ... Success!
    - Removing privileges on test database...
    ... Success!

    Reloading the privilege tables will ensure that all changes made so far
    will take effect immediately.

    Reload privilege tables now? [Y/n]

Press `Y`, then *Enter*.

    ... Success!

    Cleaning up...

    All done!  If you've completed all of the above steps, your MariaDB
    installation should now be secure.

    Thanks for using MariaDB!

<br>

**Step 7** - Create a database and a user for GLPI

    mysql -u root -p

Type your password (that one that we just created on *Step 6*). If everything is running correctly until now, you should see a screen like this:

![MariaDB monitor](/images/manjaro-glpi-mariadb-connection.png)

Let's enter some commands to achieve our goal. One at a time.

    CREATE DATABASE glpi;

    CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'StrongDBPassword';

    GRANT ALL PRIVILEGES ON glpi.* TO 'glpi'@'localhost';

    FLUSH PRIVILEGES;

    SHOW DATABASES;

    EXIT;

Remember to change the expression `StrongDBPassword` to your password. Make sure that is a safe one.

![MariaDB monitor](/images/manjaro-glpi-mariadb-newDB.png)

<br>
<hr>
<br>

### PHP

**Step 1** - Installation

    sudo pacman -S php php-apache

<br>

**Step 2** - Configure the httpd.conf file (/etc/httpd/conf/httpd.conf).

    sudo nano /etc/httpd/conf/httpd.conf

Comment the `LoadModule mpm_event_module modules/mod_mpm_event.so` line

    # LoadModule mpm_event_module modules/mod_mpm_event.so

UnComment the `# LoadModule mpm_prefork_module modules/mod_mpm_prefork.so` line

    LoadModule mpm_prefork_module modules/mod_mpm_prefork.so

UnComment the `# ServerName www.example.com:80` line and change it to:

    ServerName aGoodName:80

Search for the `<Directory "/srv/http">` block of code. Modify to the following values:

    Options -Indexes
    AllowOverride All
    Require all granted

Also, add the following lines at the end of this file:

    LoadModule php_module modules/libphp.so
    AddHandler php-script php
    Include conf/extra/php_module.conf

<br>

**Step 3** - Create an `index.php` file at the Apache home directory. This step will be necessary to check if the PHP installation was successful.

    sudo nano /srv/http/index.php

Include the following lines in this brand new file:

    <?php
    phpinfo();
    ?>

<br>

**Step 4** - Restart Apache service

    sudo systemctl restart httpd

<br>

**Step 5** - On a web browser, go to http://localhost/ and check if a page like this is showed:

![PHP Info](/images/manjaro-glpi-phpinfo.png)

This mean **success!** Congratulations, we are almost there.

**IMPORTANT** - If instead of the PHP Info page you see an "Index of/" page, you will need to do one additional step. Let's edit the httpd.conf file again:

    sudo nano /etc/httpd/conf/httpd.conf

Search for DirectoryIndex. This block will be, probably, like this:

    <IfModule dir_module>
        DirectoryIndex index.html
    </IfModule>

We will add another information after `index.html`. Type `index.php`. Now, this block needs to look like this:

    <IfModule dir_module>
        DirectoryIndex index.html index.php
    </IfModule>

Save the file, restart Apache service and try to load the http://localhost/ page again. If it's alright, remove the `index.php` file that you have created.

    sudo rm /srv/http/index.php

<br>

**Step 6** - PHP extensions installation

    sudo pacman -S php-gd php-intl php-ldap php-sodium php-apcu php-imap

<br>

**Step 7** - Configure the php.ini file (/etc/php/php.ini)

    sudo nano /etc/php/php.ini

Find and UnComment the following lines:

    extension=apcu
    extension=curl
    extension=gd
    extension=iconv
    extension=mysqli
    extension=intl
    extension=exif
    extension=imap
    extension=ldap
    extension=sodium
    extension=zip
    extension=bz2
    zend_extension=opcache

Change the following lines to theses values:

    memory_limit = 64M ;
    file_uploads = on ;
    max_execution_time = 600 ;
    session.auto_start = 0 ;
    session.use_trans_sid = 0 ;

Save the file.

<br>

**Step 8** - Configure the hosts file (/etc/hosts). For this step, use the same name used in step 2 of the PHP section (`ServerName aGoodName:80`). Once you have opened this file, you will see the actual hostname of your computer. You can add more names for resolution in this file like this:

Original file

    # Host addresses
    127.0.0.1  localhost
    127.0.1.1  BRAINIAC
    ::1        localhost ip6-localhost ip6-loopback
    ff02::1    ip6-allnodes
    ff02::2    ip6-allrouters

Modified file

    # Host addresses
    127.0.0.1  localhost
    127.0.1.1  BRAINIAC
    127.0.0.1  BRAINIAC.LOCAL
    127.0.0.1  glpi.brainiac.local
    ::1        localhost ip6-localhost ip6-loopback
    ff02::1    ip6-allnodes
    ff02::2    ip6-allrouters

Save the file.

<br>

**Step 9** - Configure the httpd.conf file (/etc/httpd/conf/httpd.conf). Yes, again.

    sudo nano /etc/httpd/conf/httpd.conf

Add the following lines at the end of this file:

    <VirtualHost 127.0.0.1:80>
        DocumentRoot /srv/http/glpi
        ServerName glpi.brainiac.local
    </VirtualHost>

Save the file and restart the Apache service:

    systemctl restart httpd

<br>
<hr>
<br>

## GLPI

**Step 1** - Download the latest version of GLPI from the [GitHub page](https://github.com/glpi-project/glpi/releases). At this time, the latest version is 9.5.5.

    cd /var/http
    wget https://github.com/glpi-project/glpi/releases/download/9.5.5/glpi-9.5.5.tgz

Extract the compressed file to the Apache folder:

    sudo tar zxvf glpi-9.5.5.tgz -C /srv/http/

Change the folder owner, from your user to the Apache:

    sudo chown -R http:http /srv/http/glpi/

<br>

**Step 2** - On a web browser, go to the link that you give as ServerName at PHP Step 9. In this example, glpi.brainiac.local

![GLPI Setup](/images/majaro-glpi-install-01.png)

Select your language;

<br>

![GLPI Setup](/images/majaro-glpi-install-02.png)

Read and accept the terms of license;

<br>

![GLPI Setup](/images/majaro-glpi-install-03.png)

Choose the option "Install";

<br>

![GLPI Setup](/images/majaro-glpi-install-04.png)

Review if your environment is ok;

<br>

![GLPI Setup](/images/majaro-glpi-install-05.png)

Insert database information (Step 7 of MySQL installation):
SQL Server: localhost
SQL User: glpi
SQL Password: StrongDBPassword

<br>

![GLPI Setup](/images/majaro-glpi-install-06.png)

Select a database. In this example, glpi;

<br>

![GLPI Setup](/images/majaro-glpi-install-07.png)

Press "Continue";

<br>

![GLPI Setup](/images/majaro-glpi-install-08.png)

Choose if you want sent telemetry data and press "Continue";

<br>

![GLPI Setup](/images/majaro-glpi-install-09.png)

Press "Continue";

<br>

![GLPI Setup](/images/majaro-glpi-install-10.png)

Press "Use GLPI";

<br>

![GLPI](/images/majaro-glpi-install-11.png)

GLPI - Login page;

<br>

![GLPI](/images/majaro-glpi-install-12.png)

GLPI - Dashboard/Start page. To remove this warnings, change the default password for the users showed and remove the `install.php` file:

    sudo rm /srv/http/glpi/install/install.php

<br>

![GLPI](/images/majaro-glpi-install-13.png)

GLPI installation finished.
