# Upgrade Core Lightning 25.09.1

## Clone lightning
~~~
sudo -i
cd /tmp
git clone https://github.com/ElementsProject/lightning.git
cd lightning
git checkout v25.09.1
~~~

## Install update and rust
~~~
sudo apt-get update
sudo apt-get install -y \
  jq autoconf automake build-essential git libtool libsqlite3-dev libffi-dev \
  python3 python3-pip net-tools zlib1g-dev libsodium-dev gettext
apt install python3-pip
curl -LsSf https://astral.sh/uv/install.sh | sh

sudo apt-get install -y cargo rustfmt protobuf-compiler
~~~

## Build CLN
~~~
sudo snap install astral-uv --classic

uv sync --all-extras --all-groups --frozen
./configure
RUST_PROFILE=release uv run make
sudo RUST_PROFILE=release make install  # This will replace lightingd executable files of the system
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

# Test CLNRest
~~~
curl -k -X POST 'https://localhost:3010/v1/listfunds' -H 'Rune: <rune>'
~~~
