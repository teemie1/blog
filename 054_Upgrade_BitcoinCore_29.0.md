# Upgrade Bitcoin Core 29.0

## Download and Check files
~~~
cd /tmp
VERSION="29.0"
wget https://bitcoincore.org/bin/bitcoin-core-$VERSION/bitcoin-$VERSION-x86_64-linux-gnu.tar.gz
wget https://bitcoincore.org/bin/bitcoin-core-$VERSION/SHA256SUMS
wget https://bitcoincore.org/bin/bitcoin-core-$VERSION/SHA256SUMS.asc
sha256sum --ignore-missing --check SHA256SUMS
gpg --verify SHA256SUMS.asc

~~~
## Stop bitcoind
~~~
sudo systemctl stop bitcoind

~~~
## Installation
~~~
$ tar -xvf bitcoin-$VERSION-x86_64-linux-gnu.tar.gz
$ sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-$VERSION/bin/*
$ bitcoin-cli --version
> Bitcoin Core RPC client version v26.1.0
> Copyright (C) 2009-2023 The Bitcoin Core developers
> [...]

~~~

## Start bitcoind
~~~
sudo systemctl start bitcoind
~~~
