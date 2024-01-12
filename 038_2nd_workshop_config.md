# The Configuration Files for 2nd BTC-LN Workshop Environment.

## Prepare Basic OS for Node
Select 6$/month on Digital Ocean
~~~
# Login as root
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
sudo ufw allow 4001/tcp comment 'allow RTL from anywhere'
sudo ufw allow 80/tcp comment 'allow http from anywhere'
sudo ufw allow 443/tcp comment 'allow https from anywhere'
sudo ufw allow 8080/tcp comment 'allow restapi from anywhere'
sudo ufw logging off
sudo ufw enable
sudo apt install nginx
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
debuglevel=info
maxpendingchannels=5
# set an external IP address
externalip=node08.satsdays.com
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
# tlsextradomain=raspibolt.local
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
sudo cp /data/lnd/data/chain/bitcoin/mainnet/admin.macaroon /home/rtl/admin.macaroon
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
  "multiPass": "BTC-LN_W0rk$h0p"
  "macaroonPath": "/home/rtl"
  "configPath": "/data/lnd/lnd.conf"

  "lnServerUrl": "https://127.0.0.1:8080"
  "swapServerUrl": "https://127.0.0.1:8081"
  "boltzServerUrl": "https://127.0.0.1:9003"
~~~

## LNbits Installation

~~~

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

## RTL - CLN
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
      "lnNode": "Node 09",
      "lnImplementation": "CLN",
      "Authentication": {
        "runePath": "/home/rtl/node09_rune.txt",
        "configPath": "/data/lightningd/config"
      },
      "Settings": {
        "userPersona": "OPERATOR",
        "themeMode": "DAY",
        "themeColor": "INDIGO",
        "fiatConversion": true,
        "currencyUnit": "THB",
        "logLevel": "ERROR",
        "lnServerUrl": "https://127.0.0.1:3010",
        "enableOffers": true
      }
    }
  ],
  "defaultNodeIndex": 1,

~~~
