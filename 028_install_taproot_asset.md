# Install Taproot Asset Daemon on Raspiblitz

## Download TAPD
~~~
$ cd /tmp
$ VERSION="0.3.0"
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/taproot-assets-linux-amd64-v$VERSION.tar.gz
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/manifest-guggero-v$VERSION.sig
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/manifest-guggero-v$VERSION.sig.ots
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/manifest-jharveyb-v$VERSION.sig
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/manifest-roasbeef-v$VERSION.sig
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/manifest-v$VERSION.txt
$ wget https://github.com/lightninglabs/taproot-assets/releases/download/v$VERSION/manifest-v$VERSION.txt.ots
$ sha256sum --check manifest-v$VERSION.txt --ignore-missing
$ ots --no-cache verify manifest-guggero-v$VERSION.sig.ots -f manifest-guggero-v$VERSION.sig
~~~

## Install TAPD
~~~
$ tar -xvf taproot-assets-linux-amd64-v$VERSION.tar.gz
$ sudo install -m 0755 -o root -g root -t /usr/local/bin taproot-assets-linux-amd64-v$VERSION/*
$ tapd --version
~~~

## User & Data Directory
~~~
$ sudo mkdir /mnt/hdd/tapd
$ sudo chown -R bitcoin:bitcoin /mnt/hdd/tapd
$ sudo su - bitcoin
$ ln -s /mnt/hdd/tapd /home/tapd/.tapd

~~~
## TAPD Configuration File
~~~
$ nano /mnt/hdd/tapd/tapd.conf
# Teemie: tapd configuration
# /mnt/hdd/tapd/tapd.conf

network=mainnet
debuglevel=debug
lnd.host=localhost:10009
lnd.macaroonpath=/mnt/hdd/lnd/data/chain/bitcoin/mainnet/admin.macaroon
lnd.tlspath=/mnt/hdd/lnd/tls.cert

~~~

## TAPD Autostart
~~~
$ exit
$ sudo nano /etc/systemd/system/tapd.service
# Teemie: systemd unit for tapd
# /etc/systemd/system/tapd.service

[Unit]
Description=Taproot Asset Protocol Daemon
Wants=lnd.service
After=lnd.service

[Service]

# Service execution
###################
ExecStart=/usr/local/bin/tapd
ExecStop=/usr/local/bin/tapcli --rpcserver=127.0.0.1:10029  --network=mainnet stop

# Process management
####################
Type=simple
Restart=always
RestartSec=30
TimeoutSec=240
LimitNOFILE=128000

# Directory creation and permissions
####################################
User=bitcoin

# /run/tapd
RuntimeDirectory=tapd
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
