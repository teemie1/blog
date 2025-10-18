# Install CLN Application 25.07

## Install prereq
~~~
sudo apt-get update
sudo apt-get install -y build-essential libcairo2-dev libjpeg-dev libpango1.0-dev libgif-dev librsvg2-dev libpixman-1-dev pkg-config python3
~~~


## Install latest NodeJS
~~~
sudo -iu lightningd

# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 20

# Verify the Node.js version:
node -v # Should print "v20.19.5".

# Verify npm version:
npm -v # Should print "10.8.2".

~~~

## Download CLN App
~~~
sudo -iu lightningd
wget https://github.com/ElementsProject/cln-application/archive/refs/tags/v25.07.tar.gz
tar -xzf v25.07.tar.gz
~~~

## Install and Compile
~~~
cd cln-application-25.07
npm ci
npm run build
npm prune --omit=dev
~~~
## Create rune for cln-app
~~~
lightning-cli createrune
lightning-cli showrunes
curl -k -X POST 'https://<HOST IP>:3010/v1/getinfo' -H 'Rune: <node-rune>'
~~~

## Edit start script
~~~
vi start_cln_app.sh

#!/bin/bash
export APP_PORT=2103
export APP_HOST=<IP Address>
export APP_CONNECT=REST
export APP_MODE=production
export APP_PROTOCOL=http
export APP_CONFIG_FILE=/home/lightningd/cln-application-25.07/config.json
export BITCOIN_HOST=localhost
export BITCOIN_NETWORK=bitcoin

export LIGHTNING_DATA_DIR=/home/lightningd/.lightning
export LIGHTNING_VARS_FILE=/home/lightningd/cln-application-25.07/.commando-env
export LIGHTNING_HOST=<IP Address>

export LIGHTNING_REST_PORT=3010
export LIGHTNING_REST_PROTOCOL=https
export LIGHTNING_REST_HOST=cln
export LIGHTNING_REST_CLIENT_KEY_FILE=/home/lightningd/.lightning/bitcoin/client-key.pem
export LIGHTNING_REST_CLIENT_CERT_FILE=/home/lightningd/.lightning/bitcoin/client.pem
export LIGHTNING_REST_CA_CERT_FILE=/home/lightningd/.lightning/bitcoin/ca.pem
export LIGHTNING_TLS_REJECT_UNAUTHORIZED=true

/home/lightningd/.nvm/versions/node/v20.19.5/bin/npm run start

vi .commando-env
LIGHTNING_RUNE=<RUNE>
LIGHTNING_PUBKEY=<NODE ID>

vi config.json
  {
    "unit": "SATS",
    "fiatUnit": "USD",
    "appMode": "DARK",
    "isLoading": false,
    "error": null,
    "singleSignOn": false,
    "password": ""
  }
~~~
## Edit /etc/hosts
~~~
vi /etc/hosts

<IP Address>	cln

~~~
## Configure Systemd for CLN-App
~~~
vi /etc/systemd/system/cln-app.service
# /etc/systemd/system/cln-app.service

[Unit]
Description=CLN Application
After=lightningd.service
PartOf=lightningd.service

[Service]
WorkingDirectory=/home/lightningd/cln-application-25.07/

ExecStart=/home/lightningd/cln-application-25.07/start_cln_app.sh
User=lightningd
TimeoutSec=120
RestartSec=30
StandardOutput=journal
StandardError=journal


[Install]
WantedBy=multi-user.target
~~~
## Start CLN-App
~~~
systemctl enable cln-app.service
systemctl start cln-app.service
~~~
## Open Port Firewall
~~~
sudo ufw allow 2103
~~~
Browse by web browser 
~~~
http://<IP Address>:2103
~~~
