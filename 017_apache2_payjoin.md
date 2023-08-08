# Configure Apache Reverse Proxy for Payjoin Testing



- Domain Name: mutiny.payjoin.org
- Public IP:   165.232.161.68
- Port:        4433,80

## Prepare Ubuntu OS
~~~
# install dependencies (assumes you don't have apache2 installed)
$ sudo apt install -y certbot apache2 tor

#add apache proxy modules
$ sudo a2enmod proxy
$ sudo a2enmod proxy_http
$ sudo a2enmod rewrite
$ sudo a2enmod headers
~~~

## Webserver setup
### Prepare Apache, SSL and Let’s Encrypt
Create a config file for the domain, e.g. /etc/apache2/sites-available/payjoin.conf

NOTE: don’t forget to update your actual domain name in 3 locations marked below.
~~~
#build the file
$ sudo nano /etc/apache2/sites-available/payjoin.conf

#insert the following for http:8080
<VirtualHost *:8080>
ServerName mutiny.payjoin.org
ServerAdmin admin@mutiny.payjoin.org

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
RewriteCond %{SERVER_NAME} =mutiny.payjoin.org
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

~~~
We will let Apache create/configure the https file once we obtain the SSL certificate.

Enable the web server config by creating a symlink and restarting apache2:
~~~
#build the symbolic link
$ sudo ln -s /etc/apache2/sites-available/payjoin.conf /etc/apache2/sites-enabled/payjoin.conf

#confirm the link is built
$ cd /etc/apache2/sites-enabled
$ ls -l
#you should see a listing of payjoin.conf in blue
#pointing to the sites-available payjoin reference

#restart apache
$ sudo systemctl restart apache2
~~~

$ sudo nano /etc/apache2/sites-available/payjoin-le-ssl.conf
~~~
<IfModule mod_ssl.c>
<VirtualHost *:4433>

ServerName mutiny.payjoin.org
ServerAdmin admin@mutiny.payjoin.org

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
# Some rewrite rules in this file were disabled on your HTTPS site,
# because they have the potential to create redirection loops.

# RewriteCond %{SERVER_NAME} =mutiny.payjoin.org
# RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

SSLCertificateFile /etc/letsencrypt/live/mutiny.payjoin.org/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/mutiny.payjoin.org/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf

ProxyRequests Off
ProxyPreserveHost On
<Proxy *>
AddDefaultCharset Off
Order deny,allow
Allow from all
</Proxy>
ProxyPass / http://127.0.0.1:3000/
ProxyPassReverse / http://127.0.0.1:3000/
RequestHeader set X-Forwarded-Proto "https"
RequestHeader set X-Forwarded-Port "4433"

</VirtualHost>
</IfModule>
~~~
Restart Apache2
~~~
$ sudo systemctl restart apache2
~~~


Now, visiting [https://mutiny.payjoin.org:4433](https://mutiny.payjoin.org:4433) should show your LNbits Server instance.
