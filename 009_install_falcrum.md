# Install Electrum Server (Falcrum)

## Install dependency
~~~
$ sudo apt install libssl-dev
$ sudo ufw allow 50002/tcp comment 'allow Fulcrum SSL from anywhere'

# Edit bitcoin.conf
$ sudo nano /data/bitcoin/bitcoin.conf
zmqpubhashblock=tcp://127.0.0.1:8433
$ sudo systemctl restart bitcoind
~~~

## Installation
~~~
$ cd /tmp
$ VERSION=1.9.1
$ wget https://github.com/cculianu/Fulcrum/releases/download/v$VERSION/Fulcrum-$VERSION-x86_64-linux.tar.gz
$ wget https://github.com/cculianu/Fulcrum/releases/download/v$VERSION/Fulcrum-$VERSION-x86_64-linux.tar.gz.asc
$ wget https://github.com/cculianu/Fulcrum/releases/download/v$VERSION/Fulcrum-$VERSION-x86_64-linux.tar.gz.sha256sum
$ curl https://raw.githubusercontent.com/Electron-Cash/keys-n-hashes/master/pubkeys/calinkey.txt | gpg --import
$ gpg --verify Fulcrum-$VERSION-x86_64-linux.tar.gz.asc

$ tar -xvf Fulcrum-$VERSION-x86_64-linux.tar.gz
$ sudo install -m 0755 -o root -g root -t /usr/local/bin Fulcrum-$VERSION-x86_64-linux/Fulcrum Fulcrum-$VERSION-x86_64-linux/FulcrumAdmin
$ Fulcrum --version

~~~

## Data directory
~~~
$ sudo adduser --disabled-password --gecos "" fulcrum
$ sudo adduser fulcrum bitcoin
$ sudo mkdir -p /data/fulcrum/fulcrum_db
$ sudo cp ~/fulcrum_teemie_banner.txt /data/fulcrum  # optional my teemie banner
$ sudo chown -R fulcrum:fulcrum /data/fulcrum/
$ sudo su - fulcrum
$ ln -s /data/fulcrum /home/fulcrum/.fulcrum
$ ls -la /home/fulcrum
$ cd /data/fulcrum
$ openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
$ wget https://raw.githubusercontent.com/minibolt-guide/minibolt/main/resources/fulcrum-banner.txt

~~~

## Configuration
~~~
$ nano /data/fulcrum/fulcrum.conf
# MiniBolt: fulcrum configuration
# /data/fulcrum/fulcrum.conf

## Bitcoin Core settings
bitcoind = 127.0.0.1:8332
rpccookie = /data/bitcoin/.cookie

## Admin Script settings
admin = 8000

## Fulcrum server general settings
datadir = /data/fulcrum/fulcrum_db
cert = /data/fulcrum/cert.pem
key = /data/fulcrum/key.pem
ssl = 0.0.0.0:50002
tcp = 0.0.0.0:50001
peering = false

# Set fast-sync according with your device,
# recommended: fast-sync=1/2 x RAM available e.g: 4GB RAM -> dbcache=2048
fast-sync = 2048

# Banner
banner = /data/fulcrum/fulcrum_teemie_banner.txt

$ exit
~~~

## Create systemd service
~~~
$ sudo nano /etc/systemd/system/fulcrum.service
# MiniBolt: systemd unit for Fulcrum
# /etc/systemd/system/fulcrum.service

[Unit]
Description=Fulcrum
Wants=bitcoind.service
After=bitcoind.service

StartLimitBurst=2
StartLimitIntervalSec=20

[Service]
ExecStart=/usr/local/bin/Fulcrum /data/fulcrum/fulcrum.conf
ExecStop=/usr/local/bin/FulcrumAdmin -p 8000 stop
KillSignal=SIGINT
User=fulcrum
Type=exec
TimeoutStopSec=300

[Install]
WantedBy=multi-user.target
~~~

~~~
$ sudo systemctl enable fulcrum
$ sudo journalctl -f -u fulcrum
$2 sudo systemctl start fulcrum
$2 sudo ss -tulpn | grep LISTEN | grep Fulcrum

~~~
