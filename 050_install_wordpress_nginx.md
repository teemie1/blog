# Install Wordpress with NGinx

## Install and Configure MySQL Database
~~~
$ sudo apt install mysql-server
$ sudo systemctl status mysql
$ mysql -u root -p

# CREATE DATABASE WordPress CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
# CREATE USER 'WordPressUser'@'localhost' IDENTIFIED BY 'your_password';
# FLUSH PRIVILEGES;
# EXIT;

~~~

## Install PHP
~~~
$ sudo apt install php8.1-cli php8.1-fpm php8.1-mysql php8.1-opcache php8.1-mbstring php8.1-xml php8.1-gd php8.1-curl
~~~

## Install WordPress with Nginx
~~~
sudo mkdir -p /var/www/html/sample.com
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar xf latest.tar.gz
sudo mv /tmp/wordpress/* /var/www/html/sample.com/
sudo chown -R www-data: /var/www/html/sample.com
sudo vi /etc/nginx/sites-available/wordpress.conf
~~~
~~~
# Redirect HTTP -> HTTPS
server {
    listen 80;
    server_name www.sample.com sample.com;

    include snippets/letsencrypt.conf;
    return 301 https://sample.com$request_uri;
}

# Redirect WWW -> NON-WWW
server {
    listen 443 ssl http2;
    server_name www.sample.com;

    ssl_certificate /etc/letsencrypt/live/sample.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sample.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/sample.com/chain.pem;
    include snippets/ssl.conf;

    return 301 https://sample.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sample.com;

    root /var/www/html/sample.com;
    index index.php;

    # SSL parameters
    ssl_certificate /etc/letsencrypt/live/sample.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/sample.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/sample.com/chain.pem;
    include snippets/ssl.conf;
    include snippets/letsencrypt.conf;

    # Log files
    access_log /var/log/nginx/sample.com.access.log;
    error_log /var/log/nginx/sample.com.error.log;

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }
}
~~~
~~~
sudo ln -s /etc/nginx/sites-available/sample.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
~~~
##
~~~

~~~
