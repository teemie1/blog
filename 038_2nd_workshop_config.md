# The Configuration Files for 2nd BTC-LN Workshop Environment.

## Prepare Basic OS for Node
Select 6$/month on Digital Ocean
~~~
# Login as root
NODENAME=node08
apt update
apt upgrade -y
sudo delgroup admin
sudo adduser --gecos "" admin
sudo usermod -a -G sudo,adm,cdrom,dip,plugdev,lxd admin
sudo apt update && sudo apt full-upgrade
sudo mkdir /data
sudo chown admin:admin /data
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp comment 'allow SSH from anywhere'
sudo ufw allow 9735/tcp comment 'allow lightning from anywhere'
sudo ufw allow 8889/tcp comment 'allow LNDg from anywhere'
sudo ufw allow 80/tcp comment 'allow http from anywhere'
sudo ufw allow 443/tcp comment 'allow https from anywhere'
sudo ufw allow 8080/tcp comment 'allow restapi from anywhere'
sudo ufw allow 10009/tcp comment 'allow rpc from anywhere'
sudo ufw logging off
sudo ufw enable
sudo apt install nginx
sudo apt install -y libnginx-mod-stream
sudo openssl req -x509 -nodes -newkey rsa:4096 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj "/CN=localhost" -days 3650
sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
sudo mkdir /etc/nginx/streams-available
sudo mkdir /etc/nginx/streams-enabled
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/nginx.conf
~~~

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
~~~
sudo nginx -t
sudo systemctl reload nginx
sudo apt install apt-transport-https
sudo nano /etc/apt/sources.list.d/tor.list
~~~
~~~
deb     [arch=amd64 signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main
deb-src [arch=amd64 signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main
~~~
~~~
wget -qO- https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --dearmor | tee /usr/share/keyrings/tor-archive-keyring.gpg >/dev/null
sudo apt update
sudo apt install tor deb.torproject.org-keyring
tor --version
sudo nano /etc/tor/torrc --linenumbers

# uncomment line 56:
ControlPort 9051

# uncomment line 60
CookieAuthentication 1

# Add line
CookieAuthFileGroupReadable 1

sudo systemctl reload tor
sudo ss -tulpn | grep LISTEN | grep tor

~~~

## Backup Path
~~~
sudo apt install nfs-common
sudo mkdir -p /data/backup
sudo mount node09.satsdays.com:/data/backup /data/backup
sudo mkdir /data/backup/$NODENAME
sudo chmod 777 /data/backup/$NODENAME
sudo echo "node09.satsdays.com:/data/backup        /data/backup    nfs" >> /etc/fstab
~~~

## LND Installation
~~~
cd /tmp
VERSION="0.17.3"
wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/lnd-linux-amd64-v$VERSION-beta.tar.gz
wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/manifest-v$VERSION-beta.txt
wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/manifest-roasbeef-v$VERSION-beta.sig
sha256sum --check manifest-v$VERSION-beta.txt --ignore-missing
curl https://raw.githubusercontent.com/lightningnetwork/lnd/master/scripts/keys/roasbeef.asc | gpg --import
gpg --verify manifest-roasbeef-v$VERSION-beta.sig manifest-v$VERSION-beta.txt
tar -xvf lnd-linux-amd64-v$VERSION-beta.tar.gz
sudo install -m 0755 -o root -g root -t /usr/local/bin lnd-linux-amd64-v$VERSION-beta/*
lnd --version
sudo adduser --disabled-password --gecos "" lnd
sudo usermod -a -G debian-tor lnd
sudo adduser admin lnd
sudo mkdir /data/lnd
sudo chown -R lnd:lnd /data/lnd

sudo su - lnd
ln -s /data/lnd /home/lnd/.lnd
ls -la
nano /data/lnd/password.txt

# Fill "BTC-LN_W0rk$h0p

chmod 600 /data/lnd/password.txt
nano /data/lnd/lnd.conf
~~~

~~~
# lnd configuration
# /data/lnd/lnd.conf

[Application Options]
alias=node08
externalip=node08.satsdays.com
backupfilepath=/data/backup/node08/channel.backup

debuglevel=info
maxpendingchannels=5
# specify an interface and port (default 9735) to listen on
listen=0.0.0.0:9735
# RPC open to all connections on Port 10009
rpclisten=0.0.0.0:10009
# REST open to all connections on Port 8080
restlisten=0.0.0.0:8080


# Password: automatically unlock wallet with the password in this file
# -- comment out to manually unlock wallet, and see RaspiBolt guide for more secure options
wallet-unlock-password-file=/data/lnd/password.txt
wallet-unlock-allow-create=true

# Automatically regenerate certificate when near expiration
tlsautorefresh=true
# Do not include the interface IPs or the system hostname in TLS certificate.
tlsdisableautofill=true
# Explicitly define any additional domain names for the certificate that will be created.
tlsextradomain=node08.satsdays.com
# tlsextradomain=raspibolt.public.domainname.com

# Channel settings
bitcoin.basefee=1000
bitcoin.feerate=1
minchansize=100000
accept-keysend=true
accept-amp=true
protocol.wumbo-channels=true
coop-close-target-confs=24

# Set to enable support for the experimental taproot channel type
protocol.simple-taproot-chans=true

# Watchtower
wtclient.active=true

# Performance
gc-canceled-invoices-on-startup=true
gc-canceled-invoices-on-the-fly=true
ignore-historical-gossip-filters=1
stagger-initial-reconnect=true

# Database
[bolt]
db.bolt.auto-compact=true
db.bolt.auto-compact-min-age=168h

[Bitcoin]
bitcoin.active=true
bitcoin.testnet=true
bitcoin.node=bitcoind

[Bitcoind]
bitcoind.rpcuser=bitcoin
bitcoind.rpcpass="BTC-LN_W0rk$h0p"
bitcoind.rpchost=node09.satsdays.com
bitcoind.zmqpubrawblock=tcp://node09.satsdays.com:28332
bitcoind.zmqpubrawtx=tcp://node09.satsdays.com:28333
bitcoind.estimatemode=ECONOMICAL

[tor]
tor.active=true
tor.v3=true
# deactivate streamisolation for hybrid-mode
tor.streamisolation=false
# activate hybrid connectivity
tor.skip-proxy-for-clearnet-targets=true
~~~
~~~
lnd
sudo su - lnd
lncli create
exit
sudo nano /etc/systemd/system/lnd.service
~~~
~~~
# RaspiBolt: systemd unit for lnd
# /etc/systemd/system/lnd.service

[Unit]
Description=LND Lightning Network Daemon
Wants=bitcoind.service
After=bitcoind.service

[Service]

# Service execution
###################
ExecStart=/usr/local/bin/lnd
ExecStop=/usr/local/bin/lncli stop

# Process management
####################
Type=simple
Restart=always
RestartSec=30
TimeoutSec=240
LimitNOFILE=128000

# Directory creation and permissions
####################################
User=lnd

# /run/lightningd
RuntimeDirectory=lightningd
RuntimeDirectoryMode=0710

# Hardening measures
####################
# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Mount /usr, /boot/ and /etc read-only for the process.
ProtectSystem=full

# Disallow the process and all of its children to gain
# new privileges through execve().
NoNewPrivileges=true

# Use a new /dev namespace only populated with API pseudo devices
# such as /dev/null, /dev/zero and /dev/random.
PrivateDevices=true

# Deny the creation of writable and executable memory mappings.
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target

~~~
~~~
sudo systemctl enable lnd
sudo systemctl start lnd
systemctl status lnd
sudo -iu admin
ln -s /data/lnd /home/admin/.lnd
exit
sudo chmod -R g+X /data/lnd/data/
sudo chmod g+r /data/lnd/data/chain/bitcoin/testnet/admin.macaroon
sudo vi /etc/profile
# Add line
alias lncli='lncli --network testnet'
~~~


## RTL Installation
~~~
cd ~
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install nodejs ca-certificates curl gnupg
sudo nano /etc/nginx/streams-enabled/rtl-reverse-proxy.conf
~~~

~~~
upstream rtl {
  server 127.0.0.1:3000;
}
server {
  listen 4001 ssl;
  proxy_pass rtl;
}
~~~
~~~
sudo nginx -t
sudo systemctl reload nginx
sudo adduser --disabled-password --gecos "" rtl
sudo cp /data/lnd/data/chain/bitcoin/testnet/admin.macaroon /home/rtl/admin.macaroon
sudo chown rtl:rtl /home/rtl/admin.macaroon
sudo su - rtl
curl https://keybase.io/suheb/pgp_keys.asc | gpg --import
git clone https://github.com/Ride-The-Lightning/RTL.git
cd RTL
git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
> v0.15.0
git checkout v0.15.0
git verify-tag v0.15.0
npm install --omit=dev --legacy-peer-deps
cp Sample-RTL-Config.json ./RTL-Config.json
nano RTL-Config.json
~~~
~~~
  "userPersona": "OPERATOR"
  "multiPass": "BTC-LN_W0rk$h0p"
  "macaroonPath": "/home/rtl"
  "configPath": "/data/lnd/lnd.conf"

  "lnServerUrl": "https://127.0.0.1:8080"
  "swapServerUrl": "https://127.0.0.1:8081"
  "boltzServerUrl": "https://127.0.0.1:9003"
~~~
~~~
node rtl
exit
sudo nano /etc/systemd/system/rtl.service
~~~
~~~
# RaspiBolt: systemd unit for Ride the Lightning
# /etc/systemd/system/rtl.service

[Unit]
Description=Ride the Lightning
After=lnd.service

[Service]
WorkingDirectory=/home/rtl/RTL
ExecStart=/usr/bin/node rtl
User=rtl

Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
~~~
~~~
sudo systemctl enable rtl
sudo systemctl start rtl
sudo journalctl -f -u rtl
~~~

## LNbits Installation

~~~
sudo apt update
sudo apt install software-properties-common 
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.9 python3.9-distutils
sudo nano /etc/nginx/sites-available/lnbits-reverse-proxy.conf
~~~
~~~
server {
    server_name node08.satsdays.com;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
~~~
~~~
sudo ln -s /etc/nginx/sites-available/lnbits-reverse-proxy.conf /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl reload nginx
sudo snap install certbot --classic
sudo certbot --nginx -d $NODENAME.satsdays.com
sudo adduser --disabled-password --gecos "" lnbits
sudo adduser lnbits lnd
sudo mkdir /data/lnbits
sudo chown -R lnbits:lnbits /data/lnbits
sudo su - lnbits
ln -s /data/lnd /home/lnbits/.lnd
ln -s /data/lnbits /home/lnbits/.lnbits
curl -sSL https://install.python-poetry.org | python3 -
export PATH="/home/lnbits/.local/bin:$PATH"
git clone https://github.com/lnbits/lnbits.git
cd lnbits  
git checkout 0.11.3
poetry env use python3.9
poetry install --only main
cp .env.example .env
vi .env
~~~
~~~
#LNBITS_DATA_FOLDER="/home/lnbits/.lnbits"
LNBITS_BACKEND_WALLET_CLASS=LndRestWallet
LND_REST_ENDPOINT=https://127.0.0.1:8080
LND_REST_CERT="/home/lnbits/.lnd/tls.cert"
LND_REST_MACAROON="/home/lnbits/.lnd/data/chain/bitcoin/testnet/admin.macaroon"
LNBITS_ADMIN_UI=true
LNBITS_SITE_TITLE="LNbits-Node08"
~~~
~~~
chmod 600 /home/lnbits/lnbits/.env
cd ~/lnbits
poetry run lnbits --port 5000 --host 0.0.0.0
exit
sudo nano /etc/systemd/system/lnbits.service
~~~
~~~
# RaspiBolt: systemd unit for LNbits
# /etc/systemd/system/lnbits.service

[Unit]
Description=LNbits
After=lnd.service
PartOf=lnd.service

[Service]
WorkingDirectory=/home/lnbits/lnbits

ExecStart=/home/lnbits/.local/bin/poetry run lnbits --port 5000 --host 0.0.0.0 --reload
User=lnbits
Restart=always
TimeoutSec=120
RestartSec=30
StandardOutput=journal
StandardError=journal

# Hardening measures
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true

[Install]
WantedBy=multi-user.target
~~~
~~~
sudo systemctl enable lnbits.service
sudo systemctl start lnbits.service
sudo systemctl status lnbits.service
sudo journalctl -f -u lnbits
sudo -iu lnbits
cd ~/lnbits
poetry run lnbits-cli superuser
# Record admin user id
https://node08.satsdays.com/admin?usr=[USER ID]
~~~

## LNbits 0.12 Update
~~~
sudo systemctl stop lnbits
sudo su - lnbits
cd /home/lnbits/lnbits
git fetch
git reset --hard HEAD
git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
> 0.12
git checkout 0.12.0
poetry install --only main
exit
sudo systemctl start lnbits
~~~
## LNDg Installation
~~~
cd ~
sudo apt install docker-compose
sudo ufw allow 8889/tcp comment 'allow LNDg'
sudo apt install virtualenv
sudo adduser root lnd
sudo adduser root www-data
sudo adduser www-data root
ln -s /data/lnd /root/.lnd

git clone https://github.com/cryptosharks131/lndg.git
cd lndg
mkdir data
echo "BTC-LN_W0rk\$h0p" > data/lndg-admin.txt
nano docker-compose.yaml
~~~
~~~
services:
  lndg:
    build: .
    volumes:
      - /root/.lnd:/root/.lnd:ro
      - /root/lndg/data:/lndg/data:rw
    command:
      - sh
      - -c
      - python initialize.py -net 'testnet' -server '127.0.0.1:10009' -d && supervisord && python manage.py runserver 0.0.0.0:8889
    network_mode: "host"
~~~
~~~
# Deploy the Docker image
docker-compose up -d

# Retrieve the admin password for login
nano data/lndg-admin.txt
Browse to http://node08.satsdays.com:8889
user: lndg-admin
password: in file data/lndg-admin.txt
~~~

## Rebalance-LND Installation
~~~
# Install Rebalance-LND
sudo apt-get install python3-pip
pip3 --version
sudo adduser --disabled-password --gecos "" rebalance-lnd
sudo adduser rebalance-lnd lnd
sudo su - rebalance-lnd
git clone https://github.com/C-Otto/rebalance-lnd.git
cd rebalance-lnd/
pip3 install -r requirements.txt .
echo 'export PATH=$PATH:/home/rebalance-lnd/rebalance-lnd' >> /home/rebalance-lnd/.bashrc
source /home/rebalance-lnd/.bashrc
rebalance.py
ln -s /data/lnd/ /home/rebalance-lnd/.lnd
cd ~/


~~~

## Bitcoin Core Configuration File
### /data/bitcoin/bitcoin.conf
~~~
# RaspiBolt: bitcoind configuration for testnet node

#[chain]
# main, test, signet, regtest
#chain=test
testnet=1

# [core]
sysperms=1
#blocksonly=1
txindex=1
# disable dbcache after full sync
#dbcache=512

# [wallet]
disablewallet=1

# [network]
listen=1
listenonion=1
proxy=127.0.0.1:9050
maxconnections=40
maxuploadtarget=5000
whitelist=download@127.0.0.1          # for Electrs

# [rpc]
rpcauth=bitcoin:c7041907c2c3abd7c6ec5defe168820e$156354266eb26789f6aed178f6d9d16d4a86caec96ed7ebe710d5efa3329364b
#Your password:
#BTC-LN_W0rk$h0p
server=1

# [zeromq]
zmqpubrawblock=tcp://0.0.0.0:28332
zmqpubrawtx=tcp://0.0.0.0:28333

# Options specific to chain, rpcport set to default values
[main]
# rpcport=8332
#bind=0.0.0.0
#rpcbind=0.0.0.0

[test]
rpcport=18332
bind=0.0.0.0
rpcbind=0.0.0.0
rpcallowip=0.0.0.0/0

[signet]
# rpcport=38332

[regtest]
# rpcport=18443
~~~

## Core Lightning Configuration File

### Install prereq for clnrest
~~~
sudo -iu lightningd
pip install -r /usr/libexec/c-lightning/plugins/clnrest/requirements.txt
~~~

### /data/lightningd/config
~~~
# MiniBolt: cln configuration
# /home/lightningd/.lightning/config

alias=node09
rgb=FFA500
network=testnet
log-file=/data/lightningd/cln.log
log-level=debug
# for admin to interact with lightning-cli
rpc-file-mode=0660

# default fees and channel min size
fee-base=1000
fee-per-satoshi=1000
min-capacity-sat=100000

## optional
# wumbo channels
large-channels
# channel confirmations needed
funding-confirms=1
# autoclean (86400=daily)
autocleaninvoice-cycle=86400
autocleaninvoice-expired-by=86400

# Bitcoin Core Connection
bitcoin-rpcuser=bitcoin
bitcoin-rpcpassword=BTC-LN_W0rk$h0p
bitcoin-rpcconnect=node09.satsdays.com
bitcoin-rpcport=18332



# network
proxy=127.0.0.1:9050
addr=statictor:127.0.0.1:9051/torport=9735
always-use-proxy=false

# CLEARNET
bind-addr=0.0.0.0:9735
announce-addr=node09.satsdays.com:9735

# Plugin
clnrest-port=3010
clnrest-protocol=https
clnrest-host=127.0.0.1

experimental-offers
experimental-splicing
experimental-dual-fund
experimental-anchors

~~~
### Service File for CLN
~~~
# MiniBolt: systemd unit for lightningd
# /etc/systemd/system/lightningd.service

[Unit]
Description=Core Lightning daemon
Requires=bitcoind.service
After=bitcoind.service
Requires=postgresql.service
After=postgresql.service
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/bin/sh -c '/usr/bin/lightningd \
                       --conf=/data/lightningd/config \
                       --daemon \
                       --pid-file=/run/lightningd/lightningd.pid'

ExecStop=/bin/sh -c '/usr/bin/lightning-cli stop'

RuntimeDirectory=lightningd

User=lightningd

# process management
Type=simple
PIDFile=/run/lightningd/lightningd.pid
Restart=on-failure
TimeoutSec=240
RestartSec=30

# hardening measures
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true

[Install]
WantedBy=multi-user.target
~~~


## certbot snap install
~~~
sudo apt install snapd
sudo apt remove certbot
sudo snap install --classic certbot
~~~

## LNbits - CLN
/home/lnbits/lnbits/.env
~~~
LNBITS_BACKEND_WALLET_CLASS=CoreLightningWallet
CORELIGHTNING_RPC="/home/lnbits/.lightning/testnet/lightning-rpc"

~~~

## RTL - Node09 
### /home/rtl/RTL/RTL-Config.json
~~~
{
  "port": "3000",
  "SSO": {
    "rtlSSO": 0,
    "rtlCookiePath": "",
    "logoutRedirectLink": ""
  },
  "nodes": [
    {
      "index": 1,
      "lnNode": "Node 01",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node01"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node01.satsdays.com:8080"
      }
    },
    {
      "index": 2,
      "lnNode": "Node 02",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node02"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node02.satsdays.com:8080"
      }
    },
    {
      "index": 3,
      "lnNode": "Node 03",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node03"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node03.satsdays.com:8080"
      }
    },
    {
      "index": 4,
      "lnNode": "Node 04",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node04"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node04.satsdays.com:8080"
      }
    },
      {
      "index": 5,
      "lnNode": "Node 05",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node05"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node05.satsdays.com:8080"
      }
    },
      {
      "index": 6,
      "lnNode": "Node 06",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node06"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node06.satsdays.com:8080"
      }
    },
      {
      "index": 7,
      "lnNode": "Node 07",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/data/backup/node07"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node07.satsdays.com:8080"
      }
    },
    {
      "index": 8,
      "lnNode": "Node 08",
      "lnImplementation": "LND",
      "Authentication": {
        "macaroonPath": "/home/rtl/node08"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "https://node08.satsdays.com:8080"
      }
    },
    {
      "index": 9,
      "lnNode": "Node 09",
      "lnImplementation": "CLN",
      "Authentication": {
        "runePath": "/home/rtl/node09_rune.txt",
        "configPath": "/data/lightningd/config"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "YELLOW",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "lnServerUrl": "https://127.0.0.1:3010",
        "enableOffers": true,
        "unannouncedChannels": false
      }
    },
    {
      "index": 10,
      "lnNode": "Node 10",
      "lnImplementation": "ECL",
      "Authentication": {
        "lnApiPassword": "eclair"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "TEAL",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "unannouncedChannels": false,
        "lnServerUrl": "http://node10.satsdays.com:8080"
      }
    },
    {
      "index": 11,
      "lnNode": "Node 11",
      "lnImplementation": "CLN",
      "Authentication": {
        "runePath": "/home/rtl/node11_rune.txt",
        "configPath": "/data/lightningd2/config"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "YELLOW",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "lnServerUrl": "https://127.0.0.1:3011",
        "enableOffers": true,
        "unannouncedChannels": false
      }
    }
  ],
  "defaultNodeIndex": 9,
  "multiPassHashed": "4ef8cf6aa9420c80d69b08acad2002162fe0ee235790b09e20d12b77cd806e44"
}
~~~
### Create rune
~~~
lightning-cli createrune
lightning-cli showrunes
~~~
### /home/rtl/node09_rune.txt
~~~
LIGHTNING_RUNE="5rNbZkJ27SVhuLQduWHfMGSvcGH_Zc_tyqFQJ89FTnM9MA=="
~~~
### /home/rtl/node11_rune.txt
~~~
LIGHTNING_RUNE="LfyhLUsorgW4I-PonB2fvvbCEVKsNWyamJ3_8fwHnuE9MA=="
~~~


## Eclair Configuration File
~~~
eclair.chain=testnet
eclair.server.public-ips=[146.190.111.127]
eclair.server.binding-ip=0.0.0.0
eclair.node-alias=node10
eclair.node-color=49daaa
eclair.server.port=9735
eclair.api.enabled=true
eclair.api.binding-ip=0.0.0.0
eclair.api.port=8080
eclair.api.password="eclair"
eclair.tor.enabled=true
eclair.tor.auth=safecookie
eclair.socks5.enabled=true
eclair.bitcoind.rpcport=18332
eclair.bitcoind.rpcuser="bitcoin"
eclair.bitcoind.rpcpassword="BTC-LN_W0rk$h0p"
eclair.bitcoind.host=node09.satsdays.com
eclair.bitcoind.zmqblock="tcp://node09.satsdays.com:28332"
eclair.bitcoind.zmqtx="tcp://node09.satsdays.com:28333"
eclair.bitcoind.wallet="teemie"
~~~

## LNbits - Eclair
~~~
#For more information on .env files, their content and format: https://pypi.org/project/python-dotenv/

######################################
########## Funding Source ############
######################################

# which fundingsources are allowed in the admin ui
LNBITS_ALLOWED_FUNDING_SOURCES="VoidWallet, FakeWallet, CoreLightningWallet, CoreLightningRestWallet, LndRestWallet, EclairWallet, LndWallet, LnTipsWallet, LNPayWallet, LNbitsWallet, AlbyWallet, OpenNodeWallet"

LNBITS_BACKEND_WALLET_CLASS=EclairWallet
# VoidWallet is just a fallback that works without any actual Lightning capabilities,
# just so you can see the UI before dealing with this file.

# Invoice expiry for LND, CLN, Eclair, LNbits funding sources
LIGHTNING_INVOICE_EXPIRY=3600

# Set one of these blocks depending on the wallet kind you chose above:

# ClicheWallet
CLICHE_ENDPOINT=ws://127.0.0.1:12000

# SparkWallet
SPARK_URL=http://localhost:9737/rpc
SPARK_TOKEN=myaccesstoken

# CoreLightningWallet
CORELIGHTNING_RPC="/home/bob/.lightning/bitcoin/lightning-rpc"

# CoreLightningRestWallet
CORELIGHTNING_REST_URL=http://127.0.0.1:8185/
CORELIGHTNING_REST_MACAROON="/path/to/clnrest/access.macaroon"  # or BASE64/HEXSTRING
CORELIGHTNING_REST_CERT="/path/to/clnrest/tls.cert"

# LnbitsWallet
LNBITS_ENDPOINT=https://legend.lnbits.com
LNBITS_KEY=LNBITS_ADMIN_KEY

# LndWallet
LND_GRPC_ENDPOINT=127.0.0.1
LND_GRPC_PORT=10009
LND_GRPC_CERT="/home/bob/.lnd/tls.cert"
LND_GRPC_MACAROON="/home/bob/.lnd/data/chain/bitcoin/mainnet/admin.macaroon"  # or HEXSTRING
# To use an AES-encrypted macaroon, set
# LND_GRPC_MACAROON="eNcRyPtEdMaCaRoOn"

# LndRestWallet
LND_REST_ENDPOINT=https://127.0.0.1:8080/
LND_REST_CERT="/home/bob/.lnd/tls.cert"
LND_REST_MACAROON="/home/bob/.lnd/data/chain/bitcoin/mainnet/admin.macaroon"  # or HEXSTRING
# To use an AES-encrypted macaroon, set
# LND_REST_MACAROON_ENCRYPTED="eNcRyPtEdMaCaRoOn"

# LNPayWallet
LNPAY_API_ENDPOINT=https://api.lnpay.co/v1/
# Secret API Key under developers tab
LNPAY_API_KEY=LNPAY_API_KEY
# Wallet Admin in Wallet Access Keys
LNPAY_WALLET_KEY=LNPAY_ADMIN_KEY

# AlbyWallet
ALBY_API_ENDPOINT=https://api.getalby.com/
ALBY_ACCESS_TOKEN=ALBY_ACCESS_TOKEN

# OpenNodeWallet
OPENNODE_API_ENDPOINT=https://api.opennode.com/
OPENNODE_KEY=OPENNODE_ADMIN_KEY

# FakeWallet
FAKE_WALLET_SECRET="ToTheMoon1"
LNBITS_DENOMINATION=sats

# EclairWallet
ECLAIR_URL=http://127.0.0.1:8080
ECLAIR_PASS=eclair

# LnTipsWallet
# Enter /api in LightningTipBot to get your key
LNTIPS_API_KEY=LNTIPS_ADMIN_KEY
LNTIPS_API_ENDPOINT=https://ln.tips

######################################
########### Admin Settings ###########
######################################

# Enable Admin GUI, available for the first user in LNBITS_ADMIN_USERS if available
# Warning: Enabling this will make LNbits ignore this configuration file. Your settings will
# be stored in your database and you will be able to change them only through the Admin UI.
# Disable this to make LNbits use this config file again.
LNBITS_ADMIN_UI=true
SUPER_USER=b4243c86d5b34969baea0cca463af07a

# Change theme
LNBITS_SITE_TITLE="LNbits-Node10"
LNBITS_SITE_TAGLINE="free and open-source lightning wallet"
LNBITS_SITE_DESCRIPTION="Some description about your service, will display if title is not 'LNbits'"
# Choose from bitcoin, mint, flamingo, freedom, salvador, autumn, monochrome, classic, cyber
LNBITS_THEME_OPTIONS="classic, bitcoin, flamingo, freedom, mint, autumn, monochrome, salvador, cyber"
# LNBITS_CUSTOM_LOGO="https://lnbits.com/assets/images/logo/logo.svg"

HOST=127.0.0.1
PORT=5000

# uvicorn variable, uncomment to allow https behind a proxy
# FORWARDED_ALLOW_IPS="*"

# Server security, rate limiting ips, blocked ips, allowed ips
LNBITS_RATE_LIMIT_NO="200"
LNBITS_RATE_LIMIT_UNIT="minute"
LNBITS_ALLOWED_IPS=""
LNBITS_BLOCKED_IPS=""

# Allow users and admins by user IDs (comma separated list)
# if set new users will not be able to create accounts
LNBITS_ALLOWED_USERS=""
LNBITS_ADMIN_USERS=""
# ID of the super user. The user ID must exist.
# SUPER_USER=""

# Extensions only admin can access
LNBITS_ADMIN_EXTENSIONS="ngrok, admin"

# Disable account creation for new users
# LNBITS_ALLOW_NEW_ACCOUNTS=false

# Enable Node Management without activating the LNBITS Admin GUI
# by setting the following variables to true.
LNBITS_NODE_UI=false
LNBITS_PUBLIC_NODE_UI=false
# Enabling the transactions tab can cause crashes on large Core Lightning nodes.
LNBITS_NODE_UI_TRANSACTIONS=false

LNBITS_DEFAULT_WALLET_NAME="LNbits wallet"

# Ad space description
# LNBITS_AD_SPACE_TITLE="Supported by"
# csv ad space, format "<url>;<img-light>;<img-dark>, <url>;<img-light>;<img-dark>", extensions can choose to honor
# LNBITS_AD_SPACE="https://shop.lnbits.com/;https://raw.githubusercontent.com/lnbits/lnbits/main/lnbits/static/images/lnbits-shop-light.png;https://raw.githubusercontent.com/lnbits/lnbits/main/lnbits/static/images/lnbits-shop-dark.png"

# Hides wallet api, extensions can choose to honor
LNBITS_HIDE_API=false

# LNBITS_EXTENSIONS_MANIFESTS="https://raw.githubusercontent.com/lnbits/lnbits-extensions/main/extensions.json,https://raw.githubusercontent.com/lnbits/lnbits-extensions/main/extensions-trial.json"
# GitHub has rate-limits for its APIs. The limit can be increased specifying a GITHUB_TOKEN
# LNBITS_EXT_GITHUB_TOKEN=github_pat_xxxxxxxxxxxxxxxxxx

# Path where extensions will be installed (defaults to `./lnbits/`).
# Inside this directory the `extensions` and `upgrades` sub-directories will be created.
# LNBITS_EXTENSIONS_PATH="/path/to/some/dir"

# Extensions to be installed by default. If an extension from this list is uninstalled then it will be re-installed on the next restart.
# The extension must be removed from this list in order to not be re-installed.
LNBITS_EXTENSIONS_DEFAULT_INSTALL="tpos"

# Database: to use SQLite, specify LNBITS_DATA_FOLDER
#           to use PostgreSQL, specify LNBITS_DATABASE_URL=postgres://...
#           to use CockroachDB, specify LNBITS_DATABASE_URL=cockroachdb://...
# for both PostgreSQL and CockroachDB, you'll need to install
#   psycopg2 as an additional dependency
LNBITS_DATA_FOLDER="/home/lnbits/.lnbits"
# LNBITS_DATABASE_URL="postgres://user:password@host:port/databasename"

# the service fee (in percent)
LNBITS_SERVICE_FEE=0.0
# the wallet where fees go to
# LNBITS_SERVICE_FEE_WALLET=
# the maximum fee per transaction (in satoshis)
# LNBITS_SERVICE_FEE_MAX=1000
# disable fees for internal transactions
# LNBITS_SERVICE_FEE_IGNORE_INTERNAL=true

# value in millisats
LNBITS_RESERVE_FEE_MIN=2000
# value in percent
LNBITS_RESERVE_FEE_PERCENT=1.0

# Limit fiat currencies allowed to see in UI
# LNBITS_ALLOWED_CURRENCIES="EUR, USD"

######################################
###### Logging and Development #######
######################################

DEBUG=false
BUNDLE_ASSETS=true

# logging into LNBITS_DATA_FOLDER/logs/
ENABLE_LOG_TO_FILE=true

# https://loguru.readthedocs.io/en/stable/api/logger.html#file
LOG_ROTATION="100 MB"
LOG_RETENTION="3 months"
~~~
