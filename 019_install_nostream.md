# Install nostream for Paid Public Nostr Relay

## Prerequisite Docker setup
 - PostgreSQL 14.0
 - Redis
 - Node v18
 - Typescript


## Install Prerequisite
~~~
# Update deps & install nodejs, npm, nginx, certbot
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install nodejs npm nginx certbot python3-certbot-nginx

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
$ rm -rf /etc/nginx/sites-available/default

# Paste in new settings file contents (see heading NGINX Settings below)
$ sudo nano /etc/nginx/sites-available/default

# Restart nginx
$ sudo systemctl start nginx

# Map DNS A record to IP of VM machine (see DNS Settings below)

# Request SSL cert from letsencrypt/certbot
$ sudo certbot --nginx -d teemie1-relay.duckdns.org

~~~
## Running as a Service
~~~
$ nano /etc/systemd/system/nostream.service

# Note: replace "User=..." with your username, and
# "/home/nostr/nostream" with the directory where you cloned the repo.

[Unit]
Description=Nostr TS Relay
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=5
User=nostr
WorkingDirectory=/home/nostr/nostream
ExecStart=/home/nostr/nostream/scripts/start
ExecStop=/home/nostr/nostream/scripts/stop

[Install]
WantedBy=multi-user.target
~~~
~~~
$ systemctl enable nostream
$ systemctl start nostream
$ journalctl -fu nostream
~~~


