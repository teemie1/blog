# Upgrade Core Lightning 25.12

## Clone lightning
~~~
sudo -i
cd /tmp
rm -rf lightning
git clone --recursive https://github.com/ElementsProject/lightning.git
cd lightning
git checkout v25.12

~~~

## Install update and rust
~~~
sudo apt update
sudo apt-get install -y valgrind libpq-dev shellcheck cppcheck \
  libsecp256k1-dev lowdown
sudo apt-get install -y cargo rustfmt protobuf-compiler
~~~

## Build CLN
~~~
rustup update
cargo update --manifest-path=cln-rpc/Cargo.toml

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
