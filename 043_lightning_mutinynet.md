# Install Lightning Node in Mutinynet

## Prepare OS
~~~
# Login as root
NODENAME=node08
apt update
apt upgrade -y
sudo mkdir /data
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp comment 'allow SSH from anywhere'
sudo ufw allow 80/tcp comment 'allow http from anywhere'
sudo ufw allow 443/tcp comment 'allow https from anywhere'
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
## Install Bitcoin Core in Mutinynet
~~~
cd /tmp
wget https://XXX
sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-$VERSION/bin/bitcoin-cli bitcoin-$VERSION/bin/bitcoind
bitcoind --version
sudo adduser --gecos "" --disabled-password bitcoin
sudo adduser tee bitcoin
sudo adduser bitcoin debian-tor
sudo mkdir /data/bitcoin
sudo chown bitcoin:bitcoin /data/bitcoin
sudo su - bitcoin
ln -s /data/bitcoin /home/bitcoin/.bitcoin
ls -la
cd .bitcoin
wget https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py
python3 rpcauth.py minibolt YourPasswordB
nano /home/bitcoin/.bitcoin/bitcoin.conf
~~~
~~~
# MiniBolt: bitcoind configuration
# /home/bitcoin/.bitcoin/bitcoin.conf

## Bitcoin daemon
server=1
txindex=1

# Additional logs
debug=tor
debug=i2p

# Assign read permission to the Bitcoin group users
startupnotify=chmod g+r /home/bitcoin/.bitcoin/.cookie

# Disable debug.log
nodebuglogfile=1

# Avoid assuming that a block and its ancestors are valid,
# and potentially skipping their script verification.
# We will set it to 0, to verify all.
assumevalid=0

# Enable all compact filters
blockfilterindex=1

# Serve compact block filters to peers per BIP 157
peerblockfilters=1

# Maintain coinstats index used by the gettxoutsetinfo RPC
coinstatsindex=1

# Network
listen=1
bind=127.0.0.1

# Connect to clearnet using Tor SOCKS5 proxy
proxy=127.0.0.1:9050

# I2P SAM proxy to reach I2P peers and accept I2P connections
i2psam=127.0.0.1:7656

## Connections
rpcauth=<replace with your own auth line generated in the previous step>

# Initial block download optimizations (set dbcache size in megabytes 
# (4 to 16384, default: 300) according to the available RAM of your device,
# recommended: dbcache=1/2 x RAM available e.g: 4GB RAM -> dbcache=2048)
# Remember to comment after IBD!
dbcache=2048
blocksonly=1
~~~
~~~
chmod 640 /home/bitcoin/.bitcoin/bitcoin.conf
exit
sudo nano /etc/systemd/system/bitcoind.service
~~~
~~~
# MiniBolt: systemd unit for bitcoind
# /etc/systemd/system/bitcoind.service

[Unit]
Description=Bitcoin Core Daemon
Requires=network-online.target
After=network-online.target

[Service]
ExecStart=/usr/local/bin/bitcoind -pid=/run/bitcoind/bitcoind.pid \
                                  -conf=/home/bitcoin/.bitcoin/bitcoin.conf \
                                  -datadir=/home/bitcoin/.bitcoin
# Process management
####################
Type=exec
NotifyAccess=all
PIDFile=/run/bitcoind/bitcoind.pid

Restart=on-failure
TimeoutStartSec=infinity
TimeoutStopSec=600

# Directory creation and permissions
####################################
User=bitcoin
Group=bitcoin
RuntimeDirectory=bitcoind
RuntimeDirectoryMode=0710
UMask=0027

# Hardening measures
####################
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true
MemoryDenyWriteExecute=true
SystemCallArchitectures=native

[Install]
WantedBy=multi-user.target
~~~
~~~
sudo systemctl enable bitcoind
journalctl -fu bitcoind
sudo systemctl start bitcoind
ln -s /data/bitcoin /home/tee/.bitcoin

~~~
