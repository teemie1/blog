# Remove Fulcrum and Install Electrum Rust Server

After synchronize Fulcrum for 14 hours until it's full sync. I can connect Blue wallet to Fulcrum once time then I found the database easily corrupted. I'm surprised for the stability of it. I decided to change back to electrs.

## Remove fulcrum database and disable Fulcrum
~~~
$ sudo rm /data/fulcrum/fulcrum_db
$ sudo systemctl stop fulcrum
$ sudo systemctl disable fulcrum
~~~

## Install Electrum Rust Server
~~~
$ sudo apt update && sudo apt full-upgrade
$ sudo apt install cargo clang cmake build-essential librocksdb-dev
$ sudo nano /etc/nginx/streams-enabled/electrs-reverse-proxy.conf
~~~
~~~
upstream electrs {
  server 127.0.0.1:50001;
}
server {
  listen 50002 ssl;
  proxy_pass electrs;
}
~~~
~~~
$ sudo nginx -t
$ sudo systemctl reload nginx
~~~
## Build from source
~~~
$ cd /tmp
$ VERSION=0.10.0
$ git clone --branch v$VERSION https://github.com/romanz/electrs.git
$ cd electrs
$ curl https://romanzey.de/pgp.txt | gpg --import
$ git verify-tag v$VERSION
$ ROCKSDB_INCLUDE_DIR=/usr/include ROCKSDB_LIB_DIR=/usr/lib CARGO_NET_GIT_FETCH_WITH_CLI=true cargo build --locked --release
$ sudo install -m 0755 -o root -g root -t /usr/local/bin ./target/release/electrs
$ electrs --version
$ cd
$ rm -r /tmp/electrs
~~~
## Electrs Configuration
~~~
$ sudo adduser --disabled-password --gecos "" electrs
$ sudo adduser electrs bitcoin
$ sudo mkdir /data/electrs
$ sudo chown -R electrs:electrs /data/electrs
$ sudo su - electrs
$ nano /data/electrs/electrs.conf
~~~
~~~
# MiniBolt: electrs configuration
# /data/electrs/electrs.conf

# Bitcoin Core settings
network = "bitcoin"
cookie_file= "/data/bitcoin/.cookie"
daemon_rpc_addr = "127.0.0.1:8332"
daemon_p2p_addr = "127.0.0.1:8333"

# Electrs settings
electrum_rpc_addr = "0.0.0.0:50001"
db_dir = "/data/electrs/db"
server_banner = "Welcome to electrs (Electrum Rust Server) running on a MiniBolt node!"

# Logging
log_filters = "INFO"
timestamp = true
~~~
~~~
$ exit
~~~
## Create systemd
~~~
$ sudo nano /etc/systemd/system/electrs.service
~~~
~~~
# MiniBolt: systemd unit for electrs
# /etc/systemd/system/electrs.service

[Unit]
Description=Electrs
Wants=bitcoind.service
After=bitcoind.service

[Service]
ExecStart=/usr/local/bin/electrs --conf /data/electrs/electrs.conf --skip-default-conf-files
Type=simple
TimeoutSec=300
KillMode=process
User=electrs
RuntimeDirectory=electrs
RuntimeDirectoryMode=0710
PrivateTmp=true
ProtectSystem=full
ProtectHome=true
PrivateDevices=true
MemoryDenyWriteExecute=true

[Install]
WantedBy=multi-user.target
~~~
~~~
$ sudo systemctl enable electrs
$ sudo journalctl -f -u electrs
$2 sudo systemctl start electrs
$2 sudo ss -tulpn | grep LISTEN | grep electrs
~~~
