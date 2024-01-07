# The Configuration Files for 2nd BTC-LN Workshop Environment.

## Bitcoin Core Configuration File
### /data/bitcoin/bitcoin.conf
~~~
# RaspiBolt: bitcoind configuration for testnet node

#[chain]
# main, test, signet, regtest
#chain=test
testnet=1

# [core]
sysperms=1
#blocksonly=1
txindex=1
# disable dbcache after full sync
#dbcache=512

# [wallet]
disablewallet=1

# [network]
listen=1
listenonion=1
proxy=127.0.0.1:9050
maxconnections=40
maxuploadtarget=5000
whitelist=download@127.0.0.1          # for Electrs

# [rpc]
rpcauth=bitcoin:c7041907c2c3abd7c6ec5defe168820e$156354266eb26789f6aed178f6d9d16d4a86caec96ed7ebe710d5efa3329364b
#Your password:
#BTC-LN_W0rk$h0p
server=1

# [zeromq]
zmqpubrawblock=tcp://0.0.0.0:28332
zmqpubrawtx=tcp://0.0.0.0:28333

# Options specific to chain, rpcport set to default values
[main]
# rpcport=8332
#bind=0.0.0.0
#rpcbind=0.0.0.0

[test]
rpcport=18332
bind=0.0.0.0
rpcbind=0.0.0.0
rpcallowip=0.0.0.0/0

[signet]
# rpcport=38332

[regtest]
# rpcport=18443
~~~

## Core Lightning Configuration File
### /data/lightningd/config
~~~
# MiniBolt: cln configuration
# /home/lightningd/.lightning/config

alias=node09
rgb=FFA500
network=testnet
log-file=/data/lightningd/cln.log
log-level=debug
# for admin to interact with lightning-cli
rpc-file-mode=0660

# default fees and channel min size
fee-base=1000
fee-per-satoshi=1
min-capacity-sat=100000

## optional
# wumbo channels
large-channels
# channel confirmations needed
funding-confirms=1
# autoclean (86400=daily)
autocleaninvoice-cycle=86400
autocleaninvoice-expired-by=86400

# Bitcoin Core Connection
bitcoin-rpcuser=bitcoin
bitcoin-rpcpassword=BTC-LN_W0rk$h0p
bitcoin-rpcconnect=node09.satsdays.com
bitcoin-rpcport=18332



# network
proxy=127.0.0.1:9050
addr=statictor:127.0.0.1:9051/torport=9735
always-use-proxy=false

# CLEARNET
bind-addr=0.0.0.0:9735
announce-addr=node09.satsdays.com:9735

~~~
