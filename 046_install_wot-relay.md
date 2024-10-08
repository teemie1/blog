# Install Web of trust relay (wot-relay)

From https://github.com/bitvora/wot-relay


## Install Golang
~~~
sudo apt install snapd
sudo snap install go --classic
go version
~~~

## Install wot-relay
~~~
sudo adduser --disabled-password --gecos "" wot
sudo usermod -a -G tee wot
sudo adduser tee wot
sudo -iu wot
git clone https://github.com/bitvora/wot-relay.git
cd wot-relay
cp .env.example .env
vi .env
~~~
Edit environment parameters
~~~
RELAY_NAME="Wot.Siamstr.Com"
RELAY_PUBKEY="11efc77b2bb7bf4eaad4e1e652b2025718b92c7c84c4fa67d2f05684e0209913"
RELAY_DESCRIPTION="This relay is Web Of Trust Relay for Siamstr Community"
RELAY_URL="wss://wot.siamstr.com"
DB_PATH="/home/wot/wot-relay/db"
INDEX_PATH="/home/wot/wot-relay/templates/index.html"
STATIC_PATH="/home/wot/wot-relay/templates/static"
REFRESH_INTERVAL=24
MINIMUM_FOLLOWERS=3 #how many followers before they're allowed in the WoT
ARCHIVAL_SYNC="FALSE" # set to TRUE to archive every note from every person in the WoT (not recommended)
ARCHIVE_REACTIONS="FALSE" # set to TRUE to archive every reaction from every person in the WoT (not recommended)
~~~
Go build
~~~
go build -ldflags "-X main.version=$(git describe --tags --always)"
#go build
~~~

## Configure Systemd Service
~~~
sudo nano /etc/systemd/system/wot-relay.service
~~~

~~~
[Unit]
Description=WOT Relay Service
After=network.target

[Service]
User=wot
ExecStart=/home/wot/wot-relay/wot-relay
WorkingDirectory=/home/wot/wot-relay
Restart=always
EnvironmentFile=/home/wot/wot-relay/.env

[Install]
WantedBy=multi-user.target
~~~
~~~
sudo systemctl daemon-reload
sudo systemctl start wot-relay
sudo systemctl enable wot-relay
~~~

## Access relay
~~~
ufw allow 3334/tcp comment 'Allow for wot-relay'
http://localhost:3334
~~~
