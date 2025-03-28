# glpi-install
glpi process installation version 10.0.18

Using Ubuntu Server 24.04.2 LTS

----------Installation Proccess------------------

Update System
sudo apt-get update -y
sudo apt-get upgrade -y

-----------Install Apache for the web server----------------

Sudo apt install apache2 -y

----------INSTALL DB-------------------
apt install mariadb-server -y
-----------
mysql_secure_installation

Then ENTER because we dont have any user with credencial in our database

Switch to unix_socket authentication [Y/N]N
select no because we need to have all the privilege to isntall the GLPI
Change the root password [Y/n] y
Remove annonymous users? [Y/n] Y
Remove test database and access to it? [Y/n]Y
Reload privilege tables [T/n]Y

---------------Create a directory for the configuration of the GLPI-------------
mkdir /etc/glpi
mkdir /var/log/glpi

------------------Install GLPI------------

wget https://github.com/glpi-project/glpi/releases/download/10.0.18/glpi-10.0.18.tgz
tar xvf glpi-10.0.18.tgz
apt install php -y
apt install -y php-{mbstring,curl,gd,xml,intl,ldap,apcu,xmlrpc,cas,zip,bz2}
apt install php-mysql -y

------------------Create Database for GLPI----------------------
mysql -u root -p
MariaDB> create database glpi charset utf8mb4 collate utf8mb4_unicode_ci;
MariaDB> create user (user)@localhost identified by '(password)*';
MariaDB> grant all privileges on glpi.* to glpi_user@localhost;
MariaDB> grant select on mysql.time_zone_name to glpi_user@localhost;

-------------------Configure GLPI-----------------------------
mv glpi /var/www/
cd /var/www/glpi
mv files/ /var/lib/glpi
nano -l inc/downstream.php

------------------------------------------------------------
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
        require_once GLPI_CONFIG_DIR . '/local_define.php';
}
------------------------------------------------------------

nano /etc/glpi/local_define.php

------------------------------------------------------------
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');
------------------------------------------------------------

rm -R -f config/
chown -R www-data: /var/www/glpi /etc/glpi/ /var/lib/glpi/ /var/log/glpi/

nano /etc/apache2/sites-enabled/000-default.conf

------------------------------------------------------------
<VirtualHost *:80>

        ServerName glpi.localhost (your host name)
        DocumentRoot /var/www/glpi/public
        <Directory /var/www/glpi/public>
                Require all granted
                RewriteEngine On
                RewriteCond %{REQUEST_FILENAME} !-f
                RewriteRule ^(.*)$ index.php [QSA,L]
        </Directory>

</VirtualHost>
---------------------------------------------------------------

a2enmod rewrite
systemctl restart apache2.service
nano -l /etc/php/8.2/apache2/php.ini

#On line 1422 configure the session.cookie_httponly = on and save




