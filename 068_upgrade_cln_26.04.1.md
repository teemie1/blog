# Upgrade Core Lightning 26.04.1

## OS Upgrade & Backup existing CLN binary
~~~
sudo -i
sudo apt update
sudo apt upgrade
reboot

mkdir /tmp/clightning-v25.12
cp /usr/local/bin/lightning* /tmp/clightning-v25.12
~~~

## Clone lightning
~~~
cd /tmp
wget https://github.com/ElementsProject/lightning/releases/download/v26.04.1/clightning-v26.04.1.zip
unzip clightning-v26.04.1.zip
cd clightning-v26.04.1

sudo rm -R /usr/local/libexec/c-lightning/plugins

~~~


## Install update and rust
~~~
sudo apt-get update
sudo apt-get install -y \
  jq autoconf automake build-essential git libtool libsqlite3-dev libffi-dev \
  python3 python3-pip net-tools zlib1g-dev libsodium-dev gettext lowdown
#pip3 install --upgrade pip
curl -LsSf https://astral.sh/uv/install.sh | sh

rustup update stable
rustup default stable
~~~

## Build CLN
~~~
uv sync --all-extras --all-groups --frozen
./configure
RUST_PROFILE=release uv run make
sudo RUST_PROFILE=release make install

lightning-cli --version
~~~
## First Start CLN
~~~
vi /mnt/data/lightning/config

database-upgrade=true

systemctl daemon-reload
systemctl start lightningd
~~~
## Restart CLN
~~~
systemctl stop lnbits
systemctl stop rtl
systemctl stop lightningd

systemctl start lightningd
systemctl start lnbits
systemctl start rtl
~~~
