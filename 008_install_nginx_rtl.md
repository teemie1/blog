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

~~~


## c-lightning-Rest plugin (No need anymore, use CLNRest instead)
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
Create Runes
~~~
$ sudo -iu lightningdm
$ lightning-cli createrune
{
   "runes": [
      {
         "rune": "[RUNE]",
         "last_used": 1763194946.505493832,
         "unique_id": "0",
         "restrictions": [],
         "restrictions_as_english": ""
      }
   ]
}
# Copy [RUNE] and save to file /home/rtl/cln/rune.txt
# /home/rtl/cln/rune.txt
LIGHTNING_RUNE="[RUNE]"

~~~
Install NodeJS & RTL
~~~
$ sudo adduser --disabled-password --gecos "" rtl
$ sudo su - rtl
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
$ source ~/.bashrc
$ nvm install 22
$ node -v
# Output should be something like: v22.16.0
$ npm -v
# Output should be the bundled npm version

$ curl https://keybase.io/suheb/pgp_keys.asc | gpg --import
$ git clone https://github.com/Ride-The-Lightning/RTL.git
$ cd RTL
$ git pull
$ git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
$ git checkout v0.15.6
$ git verify-tag v0.15.6
$ npm install --omit=dev --legacy-peer-deps
$ npm ci --omit=dev --legacy-peer-deps
~~~
Configure RTL
~~~
$ sudo su - rtl
$ nano /home/rtl/RTL/RTL-Config.json
{
  "multiPass": "<yourfancypassword>",
  "port": "4000",
  "defaultNodeIndex": 1,
  "dbDirectoryPath": "/home/rtl/RTL",
  "SSO": {
    "rtlSSO": 0,
    "rtlCookiePath": "",
    "logoutRedirectLink": ""
  },
  "nodes": [
    {
      "index": 1,
      "lnNode": "cln",
      "lnImplementation": "CLN",
      "authentication": {
        "configPath": "/data/lightningdm/config",
        "runePath": "/home/rtl/cln/rune.txt"
      },
      "settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "YELLOW",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "INFO",
        "lnServerUrl": "https://127.0.0.1:3010",
        "enableOffers": true,
        "unannouncedChannels": false
      }
    },
    {
      "index": 2,
      "lnNode": "cln2",
      "lnImplementation": "CLN",
      "authentication": {
        "configPath": "/data/lightningdm2/config",
        "runePath": "/home/rtl/cln/rune2.txt"
      },
      "settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "YELLOW",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "INFO",
        "lnServerUrl": "https://127.0.0.1:3011",
        "enableOffers": true,
        "unannouncedChannels": false
      }
    }
  ]
}


$ exit

$ sudo nano /etc/systemd/system/rtl.service
# /etc/systemd/system/rtl.service

[Unit]
Description=Ride the Lightning
After=lightningdm.service
PartOf=lightningdm.service

[Service]
WorkingDirectory=/home/rtl/RTL
ExecStart=/home/rtl/.nvm/versions/node/v24.11.1/bin/node rtl
User=rtl

Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target

$ sudo systemctl start rtl
~~~

Configure nginx
~~~
$ sudo nano /etc/nginx/streams-enabled/rtl-reverse-proxy.conf
upstream rtl {
  server 127.0.0.1:4000;
}
server {
  listen 4001 ssl;
  proxy_pass rtl;
}

$ sudo nginx -t
$ sudo systemctl reload nginx
$ sudo ufw allow 4001/tcp comment 'allow Ride The Lightning SSL from anywhere'

~~~

Access RTL via your local IP: https:10.8.0.2:4001

Ref: [https://v2.minibolt.info/bonus-guides/lightning/cln](https://v2.minibolt.info/bonus-guides/lightning/cln)
Ref RTL: [https://github.com/Ride-The-Lightning/RTL/blob/master/.github/docs/Core_lightning_setup.md](https://github.com/Ride-The-Lightning/RTL/blob/master/.github/docs/Core_lightning_setup.md)
