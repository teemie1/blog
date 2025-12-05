# Upgrade Core Lightning 25.12

## Clone lightning
~~~
sudo -i
cd /tmp
wget https://github.com/ElementsProject/lightning/releases/download/v25.12/clightning-v25.12.zip
unzip clightning-v25.12.zip
cd clightning-v25.12
~~~

## Install update and rust
~~~
sudo apt update
sudo apt install lowdown
~~~

## Build CLN
~~~
uv sync --all-extras --all-groups --frozen
./configure
RUST_PROFILE=release uv run make
sudo RUST_PROFILE=release make install  # This will replace lightingd executable files of the system

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
