# Install Wordpress with NGinx

## Install and Configure MySQL Database
~~~
$ sudo apt install mysql-server
$ mysql -V
$ sudo systemctl status mysql
$ sudo systemctl enable mysql
$ mysql -u root -p

> CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
> CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'YourStrongPassword';
> GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
> FLUSH PRIVILEGES;
> EXIT;

~~~

## Install PHP
~~~
#$ sudo apt install php8.1-cli php8.1-fpm php8.1-mysql php8.1-opcache php8.1-mbstring php8.1-xml php8.1-gd php8.1-curl
$ sudo apt install php php-cli php-common php-imap php-fpm php-snmp php-xml php-zip php-mbstring php-curl php-mysql php-gd php-intl php-json php-imagick php-bcmath
$ php -v
~~~

## Install WordPress with Nginx
~~~
cd /tmp/ && wget https://wordpress.org/latest.zip
unzip latest.zip -d /var/www
chown -R www-data:www-data /var/www/wordpress/
mv /var/www/wordpress/wp-config-sample.php /var/www/wordpress/wp-config.php
nano /var/www/wordpress/wp-config.php
~~~
~~~
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );
/** Database username */
define( 'DB_USER', 'wordpress' );
/** Database password */
define( 'DB_PASSWORD', 'YourStrongPassword' );
~~~
~~~
sudo vi /etc/nginx/sites-available/wordpress.conf
~~~
~~~
server {
listen 80;
   server_name example.com;
   root /var/www/wordpress;
   index index.php;
   server_tokens off;
   access_log /var/log/nginx/wordpress_access.log;
   error_log /var/log/nginx/wordpress_error.log;
   client_max_body_size 64M;
location / {
   try_files $uri $uri/ /index.php?$args;
}
   location ~ \.php$ {
      fastcgi_pass  unix:/run/php/php8.3-fpm.sock;
      fastcgi_index index.php;
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include /etc/nginx/fastcgi.conf;
    }
}
~~~
~~~
sudo ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
~~~
## Open browser

Now, open your web browser and access WordPress using the URL http://example.com. 
~~~

~~~
