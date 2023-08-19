# Install nostream for Paid Public Nostr Relay

## Prerequisite Docker setup
 - Docker v20.10
 - Docker Compose v2.10


## Install Prerequisite
~~~
# Update deps & install nodejs, npm, nginx, certbot
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install nodejs npm
$ sudo apt install nginx certbot python3-certbot-nginx

# Setup new `nostream` user (don't run nostream on root)
$ sudo useradd -m -G docker nostream
# If the group `docker` doesn't exist run groupadd docker

# Set new nostream user password
$ sudo passwd nostream

# Set bash shell for nostream user
$ sudo chsh -s /bin/bash nostream

~~~

## Setup Docker + Install
~~~
# Create the keyring folder
$ sudo mkdir -p /etc/apt/keyrings

# Fetch and add it to folder
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Setup proper folder permissions
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Setup `apt` Docker repository (this is a one-liner)
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
$ sudo apt update && sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Check installation is successful by checking verions
$ docker --version
~~~
## Forward port in public VPS
80 & 443
## Setup Nginx
~~~
# Delete the default nginx settings file
$ sudo rm -rf /etc/nginx/sites-available/default

# Paste in new settings file contents (see heading NGINX Settings below)
$ sudo nano /etc/nginx/sites-available/default
server {
    server_name teemie1-relay.duckdns.org;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:8008;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}


# Restart nginx
$ sudo systemctl start nginx

# Map DNS A record to IP of VM machine (see DNS Settings below)

# Request SSL cert from letsencrypt/certbot
$ sudo certbot --nginx -d teemie1-relay.duckdns.org

~~~
## Nostream Setup
~~~
# Change to nostream user
$ sudo -i -u nostream

# Clone `nostream` repo
$ git clone https://github.com/Cameri/nostream.git

# Start the relay
$ cd nostream
$ ./scripts/start

# You want to start the relay once such that all Docker images are downloaded/built, and the default settings.yaml file is automatically copied over.

# Stop the relay (you will see the NOSTREAM logo once it's running)
Ctrl + C (you can use ./scripts/stop as well)

# Edit the settings file to your liking
# (see Settings.yaml Configuration below)

# Add local.env file to root
$ touch local.env

# Edit local.env file and add ZEBEDEE_API_KEY and SECRET
# SECRET is a 128bit random hash
$ openssl rand -hex 128
$ nano .env

SECRET="your SECRET goes here"


# Restart Nostream
$ ./scripts/start

~~~
## Settings.yaml Configuration
~~~
$ nano .nostr/settings.yaml
info:
  relay_url: wss://teemie1-relay.duckdns.org
  name: teemie1-relay.duckdns.org
  description: A teemie1 nostr relay for Thai Community.
  pubkey: 11efc77b2bb7bf4eaad4e1e652b2025718b92c7c84c4fa67d2f05684e0209913
  contact: teemie1111@gmail.com
payments:
  enabled: true
  processor: lnurl
...
paymentsProcessors:
...
  lnurl:
    invoiceURL: https://getalby.com/lnurlp/teemie1

$ ./scripts/start
~~~
## Running as a service
~~~
$ sudo nano /etc/systemd/system/nostream.service

# Note: replace "User=..." with your username, and
# "/home/nostream/nostream" with the directory where you cloned the repo.

[Unit]
Description=Nostr TS Relay
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=5
User=nostream
WorkingDirectory=/home/nostream/nostream
ExecStart=/home/nostream/nostream/scripts/start
ExecStop=/home/nostream/nostream/scripts/stop

[Install]
WantedBy=multi-user.target

sudo systemctl enable nostream
sudo systemctl start nostream
sudo journalctl -u nostream
~~~
