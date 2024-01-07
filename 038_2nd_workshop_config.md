# The Configuration Files for 2nd BTC-LN Workshop Environment.

## Prepare Basic OS for Node
Select 6$/month on Digital Ocean
~~~
# Login as root
apt update
apt upgrade
sudo adduser --gecos "" admin
sudo usermod -a -G sudo,adm,cdrom,dip,plugdev,lxd admin
sudo apt update && sudo apt full-upgrade
sudo mkdir /data
udo chown admin:admin /data
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp comment 'allow SSH from anywhere'
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
sudo su
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
