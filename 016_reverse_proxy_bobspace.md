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
~~~
## Install Necessary Program
~~~
# install dependencies (assumes you don't have apache2 installed)
sudo apt install -y certbot apache2 socat tor

#add apache proxy modules
sudo a2enmod proxy
sudo a2enmod proxy_http
~~~
## SOCAT Setup
~~~
nano /etc/systemd/system/http-to-socks-proxy@.service

#add the following to the file

[Unit]
Description=HTTP-to-SOCKS proxy
After=network.target

[Service]
EnvironmentFile=/etc/http-to-socks-proxy/%i.conf
ExecStart=/usr/bin/socat tcp4-LISTEN:${LOCAL_PORT},reuseaddr,fork,keepalive,bind=127.0.0.1 SOCKS4A:${PROXY_HOST}:${REMOTE_HOST}:${REMOTE_PORT},socksport=${PROXY_PORT}

[Install]
WantedBy=multi-user.target

#exit and save
ctrl-x, y
~~~
Create the configuration for the service in /etc/http-to-socks-proxy/btcpayserver.conf:
~~~
# create the directory
mkdir -p /etc/http-to-socks-proxy/

# create the file with the content below
nano /etc/http-to-socks-proxy/btcpayserver.conf

# replace the REMOTE_HOST and adapt the ports as needed
PROXY_HOST=127.0.0.1
PROXY_PORT=9050
LOCAL_PORT=9081
REMOTE_HOST=heregoesthebtcpayserverhiddenserviceaddress.onion
REMOTE_PORT=80
~~~
Create a symlink in /etc/systemd/system/multi-user.target.wants to enable the service and start it:
~~~
# build the symbolic link
ln -s /etc/systemd/system/http-to-socks-proxy\@.service /etc/systemd/system/multi-user.target.wants/http-to-socks-proxy\@btcpayserver.service

# start
systemctl start http-to-socks-proxy@btcpayserver

# check service status
systemctl status http-to-socks-proxy@btcpayserver

# check if tunnel is active
netstat -tulpn | grep socat
# should give something like this:
# tcp 0 0 127.0.0.1:9081 0.0.0.0:* LISTEN 951/socat
~~~
## Webserver setup
### Prepare Apache, SSL and Let’s Encrypt
Create a config file for the domain, e.g. /etc/apache2/sites-available/btcpayserver.conf

NOTE: don’t forget to update your actual domain name in 3 locations marked below.
~~~
#build the file
nano /etc/apache2/sites-available/btcpayserver.conf

#insert the following for http:80
<VirtualHost *:80>
ServerName <yourdomain>.com
ServerAdmin admin@<yourdomain>.com

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
RewriteCond %{SERVER_NAME} =<yourdomain>.com
RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>

#exit and save
ctrl-X, y
~~~
We will let Apache create/configure the https file once we obtain the SSL certificate.

Enable the web server config by creating a symlink and restarting apache2:
~~~
#build the symbolic link
ln -s /etc/apache2/sites-available/btcpayserver.conf /etc/apache2/sites-enabled/btcpayserver.conf

#confirm the link is built
cd /etc/apache2/sites-enabled
ls -l
#you should see a listing of btcpayserver.conf in blue
#pointing to the sites-available btcpayserver reference

#restart apache
systemctl restart apache2
~~~

## Obtain SSL certificate via Let’s Encrypt
Run the following command and verifications:
~~~
cd /etc/apache2/sites-available
certbot --apache -d <yourdomain>.com
#Follow the prompts to accept Terms of Service and add your admin email

#Once completed, check the directory for the SSL/https version of your config file
ls -l
#you should see two files now
-rw-r--r-- 1 root root 1090 Dec 14 21:53 btcpayserver-le-ssl.conf
-rw-r--r-- 1 root root 412 Nov 29 21:47 btcpayserver.conf
~~~
Edit the btcpayserver-le-ssl.conf file
~~~
nano /etc/apache2/sites-available/btcpayserver-le-ssl.conf

# note that <yourdomain> should be pre-filled here by the Certbot process
# when creating the SSL certificate
~~~
~~~
#you should see
<IfModule mod_ssl.c>
<VirtualHost *:443>

ServerName <yourdomain-alreadyfilled>.com
ServerAdmin admin@<yourdomain-alreadyfilled>.com

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
# Some rewrite rules in this file were disabled on your HTTPS site,
# because they have the potential to create redirection loops.

# RewriteCond %{SERVER_NAME} =<yourdomain-alreadyfilled>.com
# RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

SSLCertificateFile /etc/letsencrypt/live/<yourdomain-alreadyfilled>.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/<yourdomain-alreadyfilled>.com/privkey.pem
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

ServerName <yourdomain-alreadyfilled>.com
ServerAdmin admin@<yourdomain-alreadyfilled>.com

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

RewriteEngine on
# Some rewrite rules in this file were disabled on your HTTPS site,
# because they have the potential to create redirection loops.

# RewriteCond %{SERVER_NAME} =<yourdomain-alreadyfilled>.com
# RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]

SSLCertificateFile /etc/letsencrypt/live/<yourdomain-alreadyfilled>.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/<yourdomain-alreadyfilled>.com/privkey.pem
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
systemctl restart apache2
~~~
Now, visiting <yourdomain>.com should show your BTCPay Server instance.
