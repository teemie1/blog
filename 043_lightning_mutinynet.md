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
sudo adduser --gecos "" --disabled-password bitcoinm
sudo adduser tee bitcoinm
sudo adduser bitcoinm debian-tor
sudo mkdir /data/bitcoinm
sudo chown bitcoinm:bitcoinm /data/bitcoinm
sudo su - bitcoinm
ln -s /data/bitcoinm /home/bitcoinm/.bitcoin
ls -la
cd /tmp
wget https://github.com/benthecarman/bitcoin/releases/download/mutinynet-cat-lnhance/bitcoin-c23afab47fbe-x86_64-linux-gnu.tar.gz
tar -xvf bitcoin-c23afab47fbe-x86_64-linux-gnu.tar.gz
install -m 0755 -o bitcoinm -g bitcoinm -t /home/bitcoinm bitcoin-c23afab47fbe/bin/bitcoin-cli bitcoin-c23afab47fbe/bin/bitcoind
vi ~/.profile
# Add lines
alias bitcoin-cli='/home/bitcoinm/bitcoin-cli'
alias bitcoind='/home/bitcoinm/bitcoind'

bitcoind --version
cd ~/.bitcoin
python3 /tmp/bitcoin-c23afab47fbe/share/rpcauth/rpcauth.py tee [PASSWORD]
nano /home/bitcoinm/.bitcoin/bitcoin.conf
~~~
~~~
# Bitcoind configuration for mutinynet node

#[chain]
# main, test, signet, regtest
#chain=test
signet=1

# [core]
sysperms=1
#blocksonly=1
txindex=1
# disable dbcache after full sync
dbcache=512
blocksonly=0
prune=n

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
zmqpubrawblock=tcp://0.0.0.0:48332
zmqpubrawtx=tcp://0.0.0.0:48333

# Options specific to chain, rpcport set to default values
[main]
# rpcport=8332
#bind=0.0.0.0
#rpcbind=0.0.0.0

[test]

[signet]
# rpcport=38332
signetchallenge=512102f7561d208dd9ae99bf497273e16f389bdbd6c4742ddb8e6b216e64fa2928ad8f51ae
addnode=45.79.52.207:38333
dnsseed=0
signetblocktime=30
rpcport=38332
bind=127.0.0.1
rpcbind=127.0.0.1
fallbackfee=0.0003


[regtest]
# rpcport=18443
~~~
~~~
chmod 640 /home/bitcoinm/.bitcoin/bitcoin.conf
exit
sudo nano /etc/systemd/system/bitcoinm.service
~~~
~~~
# MiniBolt: systemd unit for bitcoinm
# /etc/systemd/system/bitcoinm.service

[Unit]
Description=Bitcoin Core Daemon
Requires=network-online.target
After=network-online.target

[Service]
ExecStart=/home/bitcoinm/bitcoind -pid=/run/bitcoinm/bitcoinm.pid \
                                  -conf=/home/bitcoinm/.bitcoin/bitcoin.conf \
                                  -datadir=/home/bitcoinm/.bitcoin
# Process management
####################
Type=exec
NotifyAccess=all
PIDFile=/run/bitcoinm/bitcoinm.pid

Restart=on-failure
TimeoutStartSec=infinity
TimeoutStopSec=600

# Directory creation and permissions
####################################
User=bitcoinm
Group=bitcoinm
RuntimeDirectory=bitcoinm
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
sudo systemctl enable bitcoinm
journalctl -fu bitcoinm
sudo systemctl start bitcoinm

~~~
## Install LND
~~~
cd /tmp
VERSION="0.18.2"
wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/lnd-linux-amd64-v$VERSION-beta.tar.gz
wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/manifest-v$VERSION-beta.txt
wget https://github.com/lightningnetwork/lnd/releases/download/v$VERSION-beta/manifest-roasbeef-v$VERSION-beta.sig
sha256sum --check manifest-v$VERSION-beta.txt --ignore-missing
curl https://raw.githubusercontent.com/lightningnetwork/lnd/master/scripts/keys/roasbeef.asc | gpg --import
gpg --verify manifest-roasbeef-v$VERSION-beta.sig manifest-v$VERSION-beta.txt
tar -xvf lnd-linux-amd64-v$VERSION-beta.tar.gz
sudo install -m 0755 -o root -g root -t /usr/local/bin lnd-linux-amd64-v$VERSION-beta/*
lnd --version
sudo adduser --disabled-password --gecos "" lndm
sudo usermod -a -G debian-tor lndm
sudo adduser tee lndm
sudo mkdir /data/lndm
sudo chown -R lndm:lndm /data/lndm

sudo su - lndm
ln -s /data/lndm /home/lndm/.lnd
ls -la
nano /data/lndm/password.txt

# Fill "BTC-LN_W0rk$h0p

chmod 600 /data/lndm/password.txt
nano /data/lndm/lnd.conf
~~~

~~~
# lnd configuration
# /data/lndm/lnd.conf

[Application Options]
alias=teemie-lnd
#backupfilepath=/data/backup/channel.backup

debuglevel=info
maxpendingchannels=5
# specify an interface and port (default 9735) to listen on
listen=0.0.0.0:9745
# RPC open to all connections on Port 10009
rpclisten=0.0.0.0:10009
# REST open to all connections on Port 8080
restlisten=0.0.0.0:8080


# Password: automatically unlock wallet with the password in this file
# -- comment out to manually unlock wallet, and see RaspiBolt guide for more secure options
wallet-unlock-password-file=/data/lndm/password.txt
wallet-unlock-allow-create=true

# Automatically regenerate certificate when near expiration
tlsautorefresh=true
# Do not include the interface IPs or the system hostname in TLS certificate.
tlsdisableautofill=true
# Explicitly define any additional domain names for the certificate that will be created.
#tlsextradomain=teemie-lnd.satsdays.com
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
bitcoin.signet=true
bitcoin.node=bitcoind

[Bitcoind]
bitcoind.rpcuser=bitcoin
bitcoind.rpcpass="BTC-LN_W0rk$h0p"
bitcoind.rpchost=127.0.0.1
bitcoind.zmqpubrawblock=tcp://127.0.0.1:48332
bitcoind.zmqpubrawtx=tcp://127.0.0.1:48333
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
sudo su - lndm
lncli create
exit
sudo nano /etc/systemd/system/lndm.service
~~~
~~~
# RaspiBolt: systemd unit for lndm
# /etc/systemd/system/lndm.service

[Unit]
Description=LND Lightning Network Daemon
Wants=bitcoinm.service
After=bitcoinm.service

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
RuntimeDirectory=lightningdm
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
sudo systemctl enable lndm
sudo systemctl start lndm
systemctl status lndm
sudo -iu tee
ln -s /data/lnd /home/tee/.lnd
exit
sudo chmod -R g+X /data/lnd/data/
sudo chmod g+r /data/lnd/data/chain/bitcoin/signet/admin.macaroon
sudo vi /etc/profile
# Add line
alias lncli='lncli --network signet'
~~~

## Install Core Lightning
~~~
cd /tmp
VERSION="v24.08.1"
$ wget https://github.com/ElementsProject/lightning/releases/download/$VERSION/clightning-$VERSION-Ubuntu-24.04.tar.xz
$ wget https://github.com/ElementsProject/lightning/releases/download/$VERSION/SHA256SUMS
$ wget https://github.com/ElementsProject/lightning/releases/download/$VERSION/SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS

# Install Core Lightning
$ cd /
$ sudo tar -xvf /tmp/clightning-$VERSION-Ubuntu-24.04.tar.xz    # this will extract lightningd binary to the system

$ sudo adduser --disabled-password --gecos "" lightningdm
$ sudo usermod -a -G bitcoinm,debian-tor lightningdm
$ sudo adduser tee lightningdm
$ sudo mkdir /data/lightningdm
$ sudo mkdir /data/lightningdm-plugins-available
$ sudo chown -R lightningdm:lightningdm /data/lightningdm
$ sudo chown -R lightningdm:lightningdm /data/lightningdm-plugins-available
$ sudo su - lightningdm
$ ln -s /data/lightningdm /home/lightningdm/.lightning
$ ln -s /data/bitcoinm /home/lightningdm/.bitcoin

~~~
### Install prerequisite
~~~
exit
sudo apt-get install python3-json5 python3-flask python3-gunicorn
sudo -iu lightningdm
pip3 install --user flask-cors flask-restx pyln-client flask-socketio gevent gevent-websocket
pip3 install --user pyln-client websockets
~~~
### Configure CLN & PostgreSQL
~~~
$ cd /home/lightningdm/.lightning
$ nano config
~~~
~~~
# MiniBolt: cln configuration
# /home/lightningdm/.lightning/config

alias=teemie-cln
rgb=FFA500
network=signet
log-file=/data/lightningdm/cln.log
log-level=info
# for admin to interact with lightning-cli
rpc-file-mode=0660

# default fees and channel min size
fee-base=1000
fee-per-satoshi=1000
min-capacity-sat=1000000

## optional
# wumbo channels
#large-channels
# channel confirmations needed
funding-confirms=2
# autoclean (86400=daily)
#autocleaninvoice-cycle=86400
#autocleaninvoice-expired-by=86400
# Bitcoin Core Connection
bitcoin-rpcuser=bitcoin
bitcoin-rpcpassword=BTC-LN_W0rk$h0p
bitcoin-rpcconnect=127.0.0.1
bitcoin-rpcport=38332

# wallet settings (PostgreSQL DB)
wallet=postgres://[DB USER]:[DB PASSWORD]@localhost:5432/lightningdb


# network
proxy=127.0.0.1:9050
addr=statictor:127.0.0.1:9051/torport=9735
always-use-proxy=false

# CLEARNET
bind-addr=0.0.0.0:9735
#announce-addr=[Public Ip]:9735

# Plugin
clnrest-port=3010
clnrest-protocol=https
clnrest-host=127.0.0.1

experimental-offers
experimental-splicing
experimental-dual-fund
experimental-anchors
~~~


~~~
$ exit
$ sudo nano /etc/systemd/system/lightningdm.service
~~~

~~~
# MiniBolt: systemd unit for lightningdm
# /etc/systemd/system/lightningdm.service

[Unit]
Description=Core Lightning daemon
Requires=bitcoinm.service
After=bitcoinm.service
Requires=postgresql.service
After=postgresql.service
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/bin/sh -c '/usr/bin/lightningd \
                       --conf=/data/lightningdm/config \
                       --daemon \
                       --pid-file=/run/lightningdm/lightningdm.pid'

ExecStop=/bin/sh -c '/usr/bin/lightning-cli stop'

RuntimeDirectory=lightningdm

User=lightningdm

# process management
Type=simple
PIDFile=/run/lightningdm/lightningdm.pid
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

~~~
$ sudo systemctl enable lightningdm.service
$ sudo systemctl start lightningdm.service
$ sudo journalctl -f -u lightningdm
~~~
