


~~~
$ lightningd --alias=teemie⚡ \
             --rgb=FFA500 \
             --bitcoin-rpcuser=umbrel \
             --bitcoin-rpcpassword='H0lASO-8vR9Q9ZGZ4VNQiwFWYPJVpp8a1UBRh3Z-2eA=' \
             --bitcoin-rpcport=8332 \
             --bitcoin-rpcconnect=10.8.0.205 \
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
             --wallet=sqlite3:///data/lightningd/bitcoin/lightningd.sqlite3:/data/lightningd/bitcoin/lightningd2.sqlite3 \
             --bind-addr=0.0.0.0:9736 \
             --announce-addr=165.232.161.68:9736 \
             --daemon 
#             --wallet=postgres://lightningusr:ch!chaK0rn9103@localhost:5432/lightningdb \
~~~
