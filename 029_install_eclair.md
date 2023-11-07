# Install Eclair Lightning Server on Raspiblitz

## Install OpenJava
Reference: [link](https://adoptium.net/installation/linux/)
~~~
$ sudo -i
$ apt install -y wget apt-transport-https
$ mkdir -p /etc/apt/keyrings
$ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
$ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
$ apt update 
$ apt install temurin-11-jdk
$ mkdir /mnt/hdd/eclair
$ chown bitcoin.bitcoin /mnt/hdd/eclair
$ ln -s /mnt/hdd/eclair /home/bitcoin/.eclair

~~~

## Download Eclair and Configuration Files
~~~
$ su - bitcoin
$ wget https://github.com/ACINQ/eclair/releases/download/v0.9.0/eclair-node-0.9.0-623f7e4-bin.zip
$ unzip ./eclair-node-0.9.0-623f7e4-bin.zip
$ nano ~/.eclair/eclair.conf
eclair.chain=mainnet
eclair.node-alias=teemie⚡-ecl
eclair.node-color=49daaa
eclair.server.port=9737
eclair.api.enabled=true
eclair.api.port=8180
eclair.api.password="eclair"
eclair.tor.enabled=true
eclair.tor.auth=safecookie
eclair.socks5.enabled=true
eclair.bitcoind.rpcport=8332
eclair.bitcoind.rpcuser="raspibolt"
eclair.bitcoind.rpcpassword="raspiblitz"
eclair.bitcoind.host=127.0.0.1
eclair.bitcoind.zmqblock="tcp://127.0.0.1:28332"
eclair.bitcoind.zmqtx="tcp://127.0.0.1:28333"
eclair.bitcoind.wallet="teemie"

# Create wallet on bitcoind
$ bitcoin-cli -named createwallet wallet_name=teemie load_on_startup=true

# install eclair-cli
$ exit
$ sudo cp /home/bitcoin/eclair-node-0.9.0-623f7e4/bin/eclair-cli /usr/local/bin
~~~

## Start Eclair for test
~~~
$ sudo su - bitcoin

# Start Eclair
$ /home/bitcoin/eclair-node-0.9.0-623f7e4/bin/eclair-node.sh

# Open another terminal windows and check status
$ eclair-cli -a localhost:8180 -p eclair getinfo

# Stop eclair
$ eclair-cli -a localhost:8180 -p eclair stop

# Add alias for eclair-cli
$ exit
$ sudo nano /etc/profile
# Add at end of file
alias eclair-cli='eclair-cli -a localhost:8180 -p eclair'
~~~

## Configure Systemd
~~~
$ sudo nano /etc/systemd/system/eclair.service
[Unit]
Description=Eclair Lightning Node
Requires=bitcoind.service
After=bitcoind.service

[Service]
ExecStart=/home/bitcoin/eclair-node-0.9.0-623f7e4/bin/eclair-node.sh
Restart=always
RestartSec=60
User=bitcoin
Group=bitcoin
Type=simple

[Install]
WantedBy=multi-user.target

$ sudo systemctl enable eclair.service
$ sudo systemctl start eclair.service
$ sudo ln -s /mnt/hdd/eclair /home/admin/.eclair
$ logout

# Login again
$ eclair-cli getinfo
$ eclair-cli connect --uri=[NODE-ID]@[ADDRESS]:[PORT]
$ eclair-cli peers


~~~
