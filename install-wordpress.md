#Install Wordpress on Ubunto 16.04

##To get started with installing WordPress, follow the steps below:
	
###**Step 1: Install Apache2 HTTP Server**

WordPress CMS requires a web server and Apache2 HTTP server is the most popular open source web server available today… To install Apache2 server, run the commands below:

	sudo apt update
	sudo apt install apache2

After installing Apache2, the commands below can be used to stop, start and enable Apache2.

	sudo systemctl stop apache2.service
	sudo systemctl start apache2.service
	sudo systemctl enable apache2.service

Find out your IP address with:

	ifconfig

it will be your __*inet*__ address. Not your 127.0.0.1 which is your localhost IP.

Go to your browser and search:

**http://your-ip-here**

If apache is running you will see an **Apache2 Ubuntu Default Page**

###Step 2: Install MariaDB Database Server

WordPress requires a database server to store its content… If you’re looking for a great open source database server, then MariaDB is a great place to start. To install MariaDB run the commands below:

	sudo apt-get install mariadb-server mariadb-client

After installing MariaDB, the commands below can be used to stop, start and enable MariaDB service

##Run these on Ubuntu 16.04 LTS

	sudo systemctl stop mysql.service
	sudo systemctl start mysql.service
	sudo systemctl enable mysql.service

Next, run the commands below to secure the database server with a root password if you were not prompted to do so during the installation…
	
	sudo mysql_secure_installation

When prompted, answer the questions below with:

>Enter current password for root (enter for none): Just press the Enter
>Set root password? [Y/n]: Y
>New password: Enter password
>Re-enter new password: Repeat password
>Remove anonymous users? [Y/n]: Y
>Disallow root login remotely? [Y/n]: Y
>Remove test database and access to it? [Y/n]:  Y
>Reload privilege tables now? [Y/n]:  Y

Now that MariaDB is installed, to test whether the database server was successfully installed, run the commands below…

	sudo mysql -u root -p

type the root password when prompted…

if successfull your will see something like this:

	MariaDB [(none)]>

next exit mariaDB with:

	exit

###Step 3: Install PHP 7.2 and Related Modules

WordPress CMS is a PHP based CMS and PHP is required… However, PHP 7.2 may not be available in Ubuntu default repositories… To run PHP 7.2 on Ubuntu 16.04 and previous, you may need to run the commands below:

	sudo apt-get install software-properties-common
	sudo add-apt-repository ppa:ondrej/php

Then update and upgrade to PHP 7.2

	sudo apt update

Next, run the commands below to install PHP 7.2 and related modules.

	sudo apt install php7.2 libapache2-mod-	php7.2 php7.2-common php7.2-mysql php7.2-gmp 	php7.2-curl php7.2-intl php7.2-mbstring php7.2-xmlrpc php7.2-gd php7.2-xml php7.2-cli php7.2-zip

After installing PHP 7.2, run the commands below to open PHP default configuration file for Apache2…

	sudo nano /etc/php/7.2/apache2/php.ini

The lines below are good settings for most PHP based CMS. Update the configuration file with these and save.

	file_uploads = On
	allow_url_fopen = On
	short_open_tag = On
	memory_limit = 256M
	upload_max_filesize = 100M
	max_execution_time = 360
	date.timezone = America/Chicago

You can google php timezones if you want a timezone more relevent to your location. You can use ctrl W to search where something is located while in nano.

Everytime you make changes to PHP configuration file, you should also restart Apache2 web server… 

	sudo systemctl restart apache2.service

Now that PHP is installed, to test whether it’s functioning, create a test file called **phpinfo.php** in Apache2 default root directory…. ( **/var/www/html/**)

	sudo nano /var/www/html/phpinfo.php

Then type the content below and save the file.

	<?php phpinfo( ); ?>

Next, open your browser and browse to the server’s hostname or IP address followed by **phpinfo.php**

*__http://localhost/phpinfo.php__*

You should see PHP default test page

***Step 4: Create WordPress Database

Now that you’ve installed all the packages that are required for WordPress to run, start configuring the servers. First run the commands below to create a blank WordPress database.

To logon to MariaDB database server, run the commands below.

	sudo mysql -u root -p

Then create a database called wordpress

	CREATE DATABASE wordpress;

Create a database user called wordpressuser with a new password. Put in your own choice of username and password.

	CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'new_password_here';

Then grant the user full access to the database.

	GRANT ALL ON wordpress.* TO 'wordpressuser'@'localhost' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;

Finally, save your changes and exit.

	FLUSH PRIVILEGES;
	EXIT;

###Step 5: Download WordPress Latest Release

To get WordPress latest release you will need to go to its official download page and get it from there

	cd /tmp
	wget https://wordpress.org/latest.tar.gz
	tar -xvzf latest.tar.gz
	sudo mv wordpress /var/www/html/wordpress

next run the commands below to set the correct permissions for the WordPress root directory and to give Apache2 control.

	sudo chown -R www-data:www-data /var/www/html/wordpress/
	sudo chmod -R 755 /var/www/html/wordpress/

###Step 6: Configure Apache2

Finally, configure Apahce2 site configuration file for WordPress. This file will control how users access WordPress content. Run the commands below to create a new configuration file called **wordpress.conf**

	sudo nano /etc/apache2/sites-available/wordpress.conf

Copy and paste the content below into the file and save it. Replace the bolded items with your own domain name and directory root location.

	<VirtualHost *:80>
     ServerAdmin admin@example.com
     DocumentRoot /var/www/html/wordpress
     ServerName **example.com**
     ServerAlias **www.example.com**

     <Directory /var/www/html/wordpress/>
          Options FollowSymlinks
          AllowOverride All
          Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined
    
     <Directory /var/www/html/wordpress/>
            RewriteEngine on
            RewriteBase /
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*) index.php [PT,L]
    </Directory>
</VirtualHost>

Save the file and exit.

###Step 7: Enable the WordPress and Rewrite Module

After configuring the VirtualHost above, enable it by running the commands below:

	sudo a2ensite wordpress.conf
	sudo a2enmod rewrite
	sudo systemctl restart apache2.service

Open your browser and go to the server domain name. You should see WordPress the setup wizard to complete.

**http://example.com/**

Then follow the on-screen instructions. Select the installation language then click Continue.

And there you have it! I hope this was helpfull for you!