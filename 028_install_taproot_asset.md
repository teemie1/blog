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
$ ln -s /mnt/hdd/lnd /home/tapd/.lnd
$ ln -s /mnt/hdd/bitcoin /home/tapd/.bitcoin

~~~
## TAPD Configuration File
~~~

~~~
