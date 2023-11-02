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
$ sudo adduser --disabled-password --gecos "" tapd
$ sudo usermod -a -G tapd,debian-tor tapd
$ sudo adduser admin tapd
$ sudo mkdir /mnt/hdd/tapd
$ sudo chown -R tapd:tapd /mnt/hdd/tapd
$ sudo su - tapd
$ ln -s /mnt/hdd/tapd /home/tapd/.tapd
$ ln -s /mnt/hdd/lnd /home/tapd/.lnd
$ ln -s /mnt/hdd/bitcoin /home/tapd/.bitcoin

~~~
## TAPD Configuration File
~~~
$ nano /mnt/hdd/tapd/tapd.conf
# RaspiBolt: lnd configuration
# /data/lnd/lnd.conf

[Application Options]
alias=YOUR_FANCY_ALIAS
debuglevel=info
maxpendingchannels=5
listen=localhost

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


~~~
