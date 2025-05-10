# Upgrade Core Lightning 25.02.1

## Clone lightning
~~~
cd 
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
