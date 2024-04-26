## How to install phpIPAM on Almalinux9.3

### Features of phpIPAM

Here are the top features of phpIPAM.

- IPv4/IPv6 IP address management
- Section / Subnet management
- Automatic free space display for subnets
- Visual subnet display
- Automatic subnet scanning / IP status checks
- PowerDNS integration
- NAT support
- VLAN management
- VRF management
- IPv4 / IPv6 calculator
- IP database search
- E-mail notifications
- Custom fields support
- Translations
- Changelogs
- RACK management
- Domain authentication (AD, LDAP, Radius)
- Per-group section/subnet permissions
- Device/device types management
- RIPE subnets import
- XLS / CVS subnets import
- IP request module
- REST API
- Locations module

phpIPAM has a number of dependencies that we’ll need to install first.

- MySQL / MariaDB database server.
- PHP / PHP-FPM
- A number of PHP extensions
- A web server – Apache / Nginx


### Step 1: Install httpd and PHP

Let’s kickoff with the installation of a web server (Apache httpd), PHP and required PHP extensions.
```angular2html
dnf -y install httpd
dnf -y install php
dnf -y install php-{mysqlnd,curl,gd,intl,pear,mbstring,gettext,gmp,json,xml,fpm}
```


Start and enable both httpd and php-fpm services.
```angular2html
systemctl enable --now httpd php-fpm
```


### Step 2: Install MariaDB Database server

Update your RHEL 8 based system:
```angular2html
dnf -y update
```


The mariadb package is available in the AppStream repository and can be installed by running the command:
```angular2html
dnf install mariadb* -y
```

Activate the mariadb service using the command below:
```angular2html
systemctl enable --now mariadb
```


Once the service is started, run the command mysql_secure_installation to harden MariaDB database server security.
```angular2html
mysql_secure_installation 
```
```angular2html
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: <ENTER NEW PASSWORD>
Re-enter new password: <CONFIRM PASSWORD>
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```


Once the installation is done, login to MySQL CLI as root user and create phpIPAM database and user.
```angular2html
mysql -u root -p
```
```
CREATE DATABASE phpipam;
GRANT ALL ON phpipam.* TO phpipam@localhost IDENTIFIED BY 'IpamStr0ngP@sswOrd';
FLUSH PRIVILEGES;
QUIT;
```

Pull the latest source code of phpIPAM from the Github repository.
```angular2html
dnf -y install git
git clone --recursive https://github.com/phpipam/phpipam.git /var/www/html/phpipam
```


Configure phpIPAM:
```angular2html
cd /var/www/html/phpipam
```


Copy config.dist.php to config.php.
```angular2html
cp config.dist.php config.php
```


Edit the file:
```angular2html
vim config.php
```
```angular2html
/**
* database connection details
******************************/
$db['host'] = 'localhost';
$db['user'] = 'phpipam';
$db['pass'] = 'IpamStr0ngP@sswOrd';
$db['name'] = 'phpipam';
$db['port'] = 3306;
```


Create an Apache httpd configuration file for phpIPAM on Alamlinux9.3
```angular2html
vim /etc/httpd/conf.d/phpipam.conf
```
Add:
```angular2html
<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot "/var/www/html/phpipam"
    ServerName phpipam.computingforgeeks.com
    ServerAlias www.phpipam.computingforgeeks.com
    <Directory "/var/www/html/phpipam">
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog "/var/log/httpd/phpipam-error_log"
    CustomLog "/var/log/httpd/phpipam-access_log" combined
</VirtualHost>
```


Change ownership of the /var/www/phpipam directory to www-data user and group.
```angular2html
chown -R apache:apache /var/www/html/
```


Validate your httpd configurations.
```angular2html
apachectl -t
```



If all looks good, enable and restart httpd service.
```angular2html
systemctl enable httpd --now
```


Open the Server domain URL on http://IP or http://www.domain.com, replace domain.com with your valid domain name.


### Web installation

1. Select “New phpipam installation“. On the next page, choose the database installation method.
2. Select MySQL import instructions. This will print the command to import SQL file.
```angular2html
mysql -u root -p phpipam < /var/www/html/phpipam/db/SCHEMA.sql
```
3. When done, click the login button.

The default Login credentials are:
```angular2html
Username: admin
Password: ipamadmin
```