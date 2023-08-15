# Starting Command for Bitcoind & Lightningd


## Bitcoind
~~~
bitcoind -datadir=/data/bitcoin -server=1 -txindex=1 -listen=1 -bind=0.0.0.0 -rpcallowip=0.0.0.0/0 -rpcbind=0.0.0.0 -rpcport=8332 -rpcuser=bitcoin -rpcpassword=password -daemon=1 
~~~


## Lightningd
~~~
/usr/bin/lightningd  --alias=teemie⚡ \
                       --rgb=FFA500 \
                       --bitcoin-rpcuser=bitcoin \
                       --bitcoin-rpcpassword=password \
                       --bitcoin-rpcport=8332 \
                       --bitcoin-rpcconnect=10.8.50.160 \
                       --network=bitcoin \
                       --log-file=/data/lightningd/cln.log \
                       --log-level=info \
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
