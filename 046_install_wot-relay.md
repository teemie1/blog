# Install Web of trust relay (wot-relay)

## Install Golang
~~~

~~~

## Install wot-relay
~~~
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
ExecStart=/path/to/wot-relay
WorkingDirectory=/path/to/wot-relay
Restart=always
EnvironmentFile=/path/to/.env

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
http://localhost:3334
~~~
