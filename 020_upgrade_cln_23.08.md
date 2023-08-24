# Upgrade Core Lightning 23.08 on NODE2

## Download Core Lightning 23.08
~~~
# Download Core Lightning Software and Checksum
$ cd /tmp
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/clightning-v23.08-Ubuntu-22.04.tar.xz
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/SHA256SUMS
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS
clightning-v23.08-Ubuntu-22.04.tar.xz: OK

$ gpg --verify SHA256SUMS.asc

~~~

## Stop, upgrade and restart Core Lightning
~~~
$ sudo -i -u lightningd
$ lightning-cli stop
$ exit

# Install Core Lightning
$ cd /
$ sudo tar -xvf /tmp/clightning-v23.08-Ubuntu-22.04.tar.xz    # this will extract lightningd binary to the system

# Restart Core Lightning
$ sudo -i -u lightningd
$ /usr/bin/lightningd  --alias=teemie⚡ \
                       --rgb=FFA500 \
                       --bitcoin-rpcuser=bitcoin \
                       --bitcoin-rpcpassword=password \
                       --bitcoin-rpcport=8332 \
                       --bitcoin-rpcconnect=10.8.50.160 \
                       --network=bitcoin \
                       --log-file=/data/lightningd/cln.log \
                       --log-level=debug \
                       --rpc-file-mode=0660 \
                       --fee-base=1000 \
                       --fee-per-satoshi=1 \
                       --min-capacity-sat=1000000 \
                       --large-channels \
                       --funding-confirms=2 \
                       --autocleaninvoice-cycle=86400 \
                       --autocleaninvoice-expired-by=86400 \
                       --wallet='postgres://lightningusr:[PASSWORD]@localhost:5432/lightningdb' \
                       --bind-addr=0.0.0.0:9736 \
                       --announce-addr=165.232.161.68:9736 \
                       --always-use-proxy=false \
                       --encrypted-hsm \
                       --daemon 
~~~
## Checking version
~~~
$ sudo -i -u lightningd
$ lightning-cli getinfo
$ lightningd --version
~~~
