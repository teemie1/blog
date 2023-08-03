# Install Reverse Proxy for BOB Space LNbits

- Domain Name: bobspace-lnbits.duckdns.org
- Public IP:   178.128.120.82

## Prepare Ubuntu OS
~~~
sudo apt-get update
sudo apt-get upgrade
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 80 comment 'Standard Webserver'
sudo ufw allow 443 comment 'SSL Webserver'
sudo ufw allow 9735 comment 'LND Main Node 1'
sudo ufw enable
sudo apt install fail2ban
sudo timedatectl set-timezone Asia/Bangkok

# install dependencies (assumes you don't have apache2 installed)
sudo apt install -y certbot apache2 socat tor
sudo apt-get install python3-certbot-apache

#add apache proxy modules
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod rewrite
sudo a2enmod headers
~~~
## SOCAT Setup
~~~
sudo nano /etc/systemd/system/http-to-socks-proxy@.service

#add the following to the file

[Unit]
Description=HTTP-to-SOCKS proxy
After=network.target

[Service]
EnvironmentFile=/etc/http-to-socks-proxy/%i.conf
ExecStart=/usr/bin/socat tcp4-LISTEN:${LOCAL_PORT},reuseaddr,fork,keepalive,bind=127.0.0.1 SOCKS4A:${PROXY_HOST}:${REMOTE_HOST}:${REMOTE_PORT},socksport=${PROXY_PORT}

[Install]
WantedBy=multi-user.target
~~~
Create the configuration for the service in /etc/http-to-socks-proxy/lnbits.conf:
~~~
# create the directory
sudo mkdir -p /etc/http-to-socks-proxy/

# create the file with the content below
sudo nano /etc/http-to-socks-proxy/lnbits.conf

# replace the REMOTE_HOST and adapt the ports as needed
PROXY_HOST=127.0.0.1
PROXY_PORT=9050
LOCAL_PORT=9081
REMOTE_HOST=ksgckoebikx53v4iarh5aops3i7arjwznmlin2j2pc7rpdgbfyeo2kyd.onion
REMOTE_PORT=80
~~~
Create a symlink in /etc/systemd/system/multi-user.target.wants to enable the service and start it:
~~~
# build the symbolic link
sudo ln -s /etc/systemd/system/http-to-socks-proxy\@.service /etc/systemd/system/multi-user.target.wants/http-to-socks-proxy\@lnbits.service

# start
sudo systemctl start http-to-socks-proxy@lnbits

# check service status
sudo systemctl status http-to-socks-proxy@lnbits

# check if tunnel is active
sudo ss -tulpn | grep socat
# should give something like this:
# tcp 0 0 127.0.0.1:9081 0.0.0.0:* 
~~~
## Webserver setup
### Prepare Apache, SSL and Let’s Encrypt
Create a config file for the domain, e.g. /etc/apache2/sites-available/lnbits.conf

NOTE: don’t forget to update your actual domain name in 3 locations marked below.
~~~
#build the file
sudo nano /etc/apache2/sites-available/lnbits.conf

#insert the following for http:80
<VirtualHost *:80>
ServerName bobspace-lnbits.duckdns.org
ServerAdmin admin@bobspace-lnbits.duckdns.org

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
RewriteCond %{SERVER_NAME} =bobspace-lnbits.duckdns.org
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

~~~
We will let Apache create/configure the https file once we obtain the SSL certificate.

Enable the web server config by creating a symlink and restarting apache2:
~~~
#build the symbolic link
sudo ln -s /etc/apache2/sites-available/lnbits.conf /etc/apache2/sites-enabled/lnbits.conf

#confirm the link is built
cd /etc/apache2/sites-enabled
ls -l
#you should see a listing of lnbits.conf in blue
#pointing to the sites-available lnbits reference

#restart apache
sudo systemctl restart apache2
~~~

## Obtain SSL certificate via Let’s Encrypt
Run the following command and verifications:
~~~
cd /etc/apache2/sites-available
sudo certbot --apache -d bobspace-lnbits.duckdns.org
#Follow the prompts to accept Terms of Service and add your admin email

#Once completed, check the directory for the SSL/https version of your config file
ls -l
#you should see two files now
-rw-r--r-- 1 root root 1090 Dec 14 21:53 lnbits-le-ssl.conf
-rw-r--r-- 1 root root 412 Nov 29 21:47 lnbits.conf
~~~
Edit the lnbits-le-ssl.conf file
~~~
sudo nano /etc/apache2/sites-available/lnbits-le-ssl.conf

# note that <yourdomain> should be pre-filled here by the Certbot process
# when creating the SSL certificate
~~~
~~~
#you should see
<IfModule mod_ssl.c>
<VirtualHost *:443>

ServerName bobspace-lnbits.duckdns.org
ServerAdmin admin@bobspace-lnbits.duckdns.org

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
# Some rewrite rules in this file were disabled on your HTTPS site,
# because they have the potential to create redirection loops.

# RewriteCond %{SERVER_NAME} =<yourdomain-alreadyfilled>.com
# RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

SSLCertificateFile /etc/letsencrypt/live/bobspace-lnbits.duckdns.org/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/bobspace-lnbits.duckdns.org/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf

</VirtualHost>
</IfModule>
~~~
Add the following text to the blank line above
~~~
ProxyRequests Off
ProxyPreserveHost On
<Proxy *>
AddDefaultCharset Off
Order deny,allow
Allow from all
</Proxy>
ProxyPass / http://127.0.0.1:9081/
ProxyPassReverse / http://127.0.0.1:9081/
RequestHeader set X-Forwarded-Proto "https"
RequestHeader set X-Forwarded-Port "443"
~~~
Your file should look like this now:
~~~
<IfModule mod_ssl.c>
<VirtualHost *:443>

ServerName bobspace-lnbits.duckdns.org
ServerAdmin admin@bobspace-lnbits.duckdns.org

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
# Some rewrite rules in this file were disabled on your HTTPS site,
# because they have the potential to create redirection loops.

# RewriteCond %{SERVER_NAME} =bobspace-lnbits.duckdns.org
# RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

SSLCertificateFile /etc/letsencrypt/live/bobspace-lnbits.duckdns.org/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/bobspace-lnbits.duckdns.org/privkey.pem
Include /etc/letsencrypt/options-ssl-apache.conf

ProxyRequests Off
ProxyPreserveHost On
<Proxy *>
AddDefaultCharset Off
Order deny,allow
Allow from all
</Proxy>
ProxyPass / http://127.0.0.1:9081/
ProxyPassReverse / http://127.0.0.1:9081/
RequestHeader set X-Forwarded-Proto "https"
RequestHeader set X-Forwarded-Port "443"

</VirtualHost>
</IfModule>
~~~
Restart Apache2
~~~
sudo systemctl restart apache2
~~~
## Install tor
~~~
$ sudo apt install apt-transport-https
$ sudo nano /etc/apt/sources.list.d/tor.list
# Add the following line list
deb     [arch=amd64 signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main
deb-src [arch=amd64 signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main

$ sudo su -
$ wget -qO- https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --dearmor | tee /usr/share/keyrings/tor-archive-keyring.gpg >/dev/null
$ exit
$ sudo apt update
$ sudo apt install tor deb.torproject.org-keyring
$ tor --version

~~~
## Tor configuration
~~~
$ sudo nano /etc/tor/torrc --linenumbers
# uncomment line 56:
ControlPort 9051

# uncomment line 60
CookieAuthentication 1

# add under line 60:
CookieAuthFileGroupReadable 1

$ sudo systemctl restart tor
$ sudo ss -tulpn | grep LISTEN | grep tor
~~~

Now, visiting [https://bobspace-lnbits.duckdns.org](https://bobspace-lnbits.duckdns.org) should show your LNbits Server instance.
