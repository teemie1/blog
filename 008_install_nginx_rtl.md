# Install Nginx and Ride The Lightning for Core Lightning

## Install and configure NGINX
~~~
$ sudo apt install nginx
# Create a self-signed SSL/TLS certificate (valid for 10 years)
$ sudo openssl req -x509 -nodes -newkey rsa:4096 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj "/CN=localhost" -days 3650
$ sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
$ sudo nano /etc/nginx/nginx.conf
~~~
Fill text to nginx.conf
~~~
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
  worker_connections 768;
}

http {
  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
  ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
  ssl_session_cache shared:HTTP-TLS:1m;
  ssl_session_timeout 4h;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;
  include /etc/nginx/sites-enabled/*.conf;
}

stream {
  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
  ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
  ssl_session_cache shared:STREAM-TLS:1m;
  ssl_session_timeout 4h;
  ssl_protocols TLSv1.2 TLSv1.3;
  ssl_prefer_server_ciphers on;
  include /etc/nginx/streams-enabled/*.conf;
}
~~~
Create a new directory for future configuration files
~~~
$ sudo mkdir /etc/nginx/streams-enabled
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo nginx -t
$ sudo systemctl reload nginx
~~~
## Install NodeJS 18 LTS
~~~
$ cd ~
$ curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
$ sudo apt install nodejs
~~~


## c-lightning-Rest plugin
~~~
$ sudo su - lightningd
$ wget https://github.com/Ride-The-Lightning/c-lightning-REST/archive/refs/tags/v0.10.5.tar.gz
$ wget https://github.com/Ride-The-Lightning/c-lightning-REST/releases/download/v0.10.5/v0.10.5.tar.gz.asc

$ curl https://keybase.io/suheb/pgp_keys.asc | gpg --import
$ gpg --verify v0.10.5.tar.gz.asc v0.10.5.tar.gz
$ tar xvf v0.10.5.tar.gz
$ cd c-lightning-REST-0.10.5
$ npm install

$ cp -r ~/c-lightning-REST-0.10.5/ /data/lightningd-plugins-available/

$ nano /data/lightningd/config
# cln-rest-plugin
plugin=/data/lightningd-plugins-available/c-lightning-REST-0.10.5/clrest.js
rest-port=3092
rest-docport=4091
rest-protocol=https

$ cd /data/lightningd-plugins-available/c-lightning-REST-0.10.5
$ cp sample-cl-rest-config.json cl-rest-config.json
$ nano cl-rest-config.json
{
  "PORT": 3092,
  "DOCPORT": 4091,
  "PROTOCOL": "https",
  "EXECMODE": "production",
  "RPCCOMMANDS": ["*"],
  "DOMAIN": "localhost"
}

$ node cl-rest.js
$ exit

$ sudo systemctl restart lightningd
~~~

## Install & Configuring RTL

Copy magaroon & Install RTL
~~~
$ sudo adduser --disabled-password --gecos "" rtl
$ sudo cp /data/lightningd-plugins-available/c-lightning-REST-0.10.5/certs/access.macaroon /home/rtl/
$ sudo chown rtl:rtl /home/rtl/access.macaroon
$ sudo su - rtl
$ curl https://keybase.io/suheb/pgp_keys.asc | gpg --import
$ git clone https://github.com/Ride-The-Lightning/RTL.git
$ cd RTL
$ git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
$ git checkout v0.14.0
$ git verify-tag v0.14.0
$ npm install --omit=dev --legacy-peer-deps
~~~
Configure RTL
~~~
$ sudo su - rtl
$ nano /home/rtl/RTL/RTL-Config.json
{
  "multiPass": "<yourfancypassword>",
  "port": "3000",
  "SSO": {
    "rtlSSO": 0,
    "rtlCookiePath": "",
    "logoutRedirectLink": ""
  },
  "nodes": [
    {
      "index": 1,
      "lnNode": "teemie⚡",
      "lnImplementation": "CLN",
      "Authentication": {
        "macaroonPath": "/home/rtl",
        "configPath": "/data/lightningd/config"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "lnServerUrl": "https://127.0.0.1:3092",
        "enableOffers": true
      }
    }
  ],
  "defaultNodeIndex": 1
}

$ exit

$ sudo nano /etc/systemd/system/rtl.service
# MiniBolt: systemd unit for Ride the Lightning
# /etc/systemd/system/rtl.service

[Unit]
Description=Ride the Lightning
After=lightningd.service
PartOf=lightningd.service

[Service]
WorkingDirectory=/home/rtl/RTL
ExecStart=/usr/bin/node rtl
User=rtl

Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target

$ sudo systemctl start rtl
~~~

Configure nginx
~~~
$ sudo nano /etc/nginx/sites-enabled/rtl-reverse-proxy.conf
server {
  listen 4001 ssl;
  error_page 497 =301 https://$host:$server_port$request_uri;

  location / {
    proxy_pass http://127.0.0.1:3000;
  }
}

$ sudo nginx -t
$ sudo systemctl reload nginx
$ sudo ufw allow 4001/tcp comment 'allow Ride The Lightning SSL from anywhere'

~~~

Access RTL via your local IP: https:10.8.0.2:4001

Ref: [https://v2.minibolt.info/bonus-guides/lightning/cln](https://v2.minibolt.info/bonus-guides/lightning/cln)
Ref RTL: [https://github.com/Ride-The-Lightning/RTL/blob/master/.github/docs/Core_lightning_setup.md](https://github.com/Ride-The-Lightning/RTL/blob/master/.github/docs/Core_lightning_setup.md)
