# Install Web of trust relay (wot-relay)

## Install Golang
~~~
sudo apt-get update && sudo apt-get -y install golang-go
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
RELAY_NAME="YourRelayName"
RELAY_PUBKEY="YourPublicKey"
RELAY_DESCRIPTION="Your relay description"
DB_PATH="/path/to/your/database"
~~~
Go build
~~~
go build
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
ExecStart=/home/wot/wot-relay
WorkingDirectory=/home/wot/wot-relay
Restart=always
EnvironmentFile=/home/wot/.env

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
