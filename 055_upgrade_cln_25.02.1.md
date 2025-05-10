# Upgrade Core Lightning 25.02.1

## Install Prereq
~~~
sudo apt-get install -y \
  jq autoconf automake build-essential git libtool libsqlite3-dev libffi-dev \
  python3 python3-pip net-tools zlib1g-dev libsodium-dev gettext
pip3 install --upgrade pip
pip3 install --user poetry
~~~
## Clone lightning
~~~
cd /tmp
git clone https://github.com/ElementsProject/lightning.git
cd lightning
git checkout v25.02.1
~~~

## Install rust
~~~
sudo apt-get install -y cargo rustfmt protobuf-compiler
~~~

## Build CLN
~~~
poetry install
./configure
RUST_PROFILE=release poetry run make
sudo RUST_PROFILE=release make install
~~~
