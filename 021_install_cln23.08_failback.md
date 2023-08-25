# Install Core Lightning 23.08 and Failback to NODE1

## Download Core Lightning 23.08
~~~
# Download Core Lightning Software and Checksum
$ cd /tmp
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/clightning-v23.08-Ubuntu-22.04.tar.xz
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/SHA256SUMS
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS
clightning-v23.08-Ubuntu-22.04.tar.xz: OK

# Download and verify key with gpg
$ wget https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/rustyrussell.txt
$ wget https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/cdecker.txt
$ wget https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/niftynei.txt
$ gpg --import rustyrussell.txt
$ gpg --import cdecker.txt
$ gpg --import niftynei.txt
$ gpg --verify SHA256SUMS.asc

# Install Core Lightning
$ cd /
$ sudo tar -xvf /tmp/clightning-v23.08-Ubuntu-22.04.tar.xz    # this will extract lightningd binary to the system
$ lightningd --version
~~~


## Prepare environment for Core Lightning
~~~
$ sudo adduser --disabled-password --gecos "" lightningd
$ sudo usermod -a -G bitcoin,debian-tor lightningd
$ sudo adduser tee lightningd
$ sudo mkdir /data/lightningd
$ sudo mkdir /data/lightningd/bitcoin
$ sudo mkdir /data/lightningd-plugins-available
$ sudo chown -R lightningd:lightningd /data/lightningd
$ sudo chown -R lightningd:lightningd /data/lightningd-plugins-available
$ sudo su - lightningd
$ ln -s /data/lightningd /home/lightningd/.lightning
$ ln -s /data/bitcoin /home/lightningd/.bitcoin

$ exit
$ ln -s /data/lightningd /home/tee/.lightning
$ sudo chmod -R g+x /data/lightningd/bitcoin/

~~~


## Configure CLN & PostgreSQL
~~~
$ sudo -i -u lightningd    # already created user for lightningd
$ cd /home/lightningd/.lightning
$ nano config
~~~


### Edit CLN Configuration File
~~~
# teemie node: cln configuration
# /home/lightningd/.lightning/config

alias=teemie⚡ 
rgb=FFA500
network=bitcoin
log-file=/data/lightningd/cln.log
log-level=debug
# for admin to interact with lightning-cli
rpc-file-mode=0660

# default fees and channel min size
fee-base=1000
fee-per-satoshi=1000
min-capacity-sat=1000000

## optional
# wumbo channels
large-channels
# channel confirmations needed
funding-confirms=2
# autoclean (86400=daily)
autocleaninvoice-cycle=86400
autocleaninvoice-expired-by=86400

# wallet settings (PostgreSQL DB)
wallet=postgres://lightningusr:[PASSWORD]@localhost:5432/lightningdb


# network
proxy=127.0.0.1:9050
addr=statictor:127.0.0.1:9051/torport=9736
always-use-proxy=false

# CLEARNET
bind-addr=0.0.0.0:9736
announce-addr=165.232.161.68:9736

~~~


## Autostart on boot
~~~
$ exit
$ sudo nano /etc/systemd/system/lightningd.service
~~~
~~~
# MiniBolt: systemd unit for lightningd
# /etc/systemd/system/lightningd.service

[Unit]
Description=Core Lightning daemon
Requires=bitcoind.service
After=bitcoind.service
Requires=postgresql.service
After=postgresql.service
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/bin/sh -c '/usr/bin/lightningd \
                       --conf=/data/lightningd/config \
                       --daemon \
                       --pid-file=/run/lightningd/lightningd.pid'

ExecStop=/bin/sh -c '/usr/bin/lightning-cli stop'

RuntimeDirectory=lightningd

User=lightningd

# process management
Type=simple
PIDFile=/run/lightningd/lightningd.pid
Restart=on-failure
TimeoutSec=240
RestartSec=30

# hardening measures
PrivateTmp=true
ProtectSystem=full
NoNewPrivileges=true
PrivateDevices=true

[Install]
WantedBy=multi-user.target
~~~
## Check PostgreSQL Database
~~~
# Check block on NODE2 (primary)
$ sudo -i -u lightningd
$ lightning-cli getinfo

# Check block on NODE1 (secondary)
$ sudo -i -u postgres
$ psql -U lightningusr --host=localhost --port=5432 "dbname=lightningdb" -t -c "SELECT max(height) from blocks;"

~~~

## Stop Core Lightning on NODE2
~~~
# Login NODE2
$ sudo -i -u lightningd
$ lightning-cli stop
$ exit
$ sudo systemctl stop postgresql
$ sudo systemctl disable postgresql
~~~

## Promote on NODE1
~~~
# Login NODE1
$ sudo -i -u postgres

# Check cluster version & name
$ pg_lsclusters

# Check cluster configuration
$ pg_config

# In case for fast failover
$ pg_ctlcluster 14 main promote

# In case for prepare replication to NODE2
$ rm /var/lib/postgresql/14/main/standby.signal
# Change configuration file
$ nano /etc/postgresql/14/main/postgresql.conf
# Comment out
#primary_conninfo = 'host=10.8.1.4 port=5432 user=lightningusr password=''[PASSWORD]'' application_name=lightningd dbname=replication'
#primary_slot_name = 'node_a_slot'
# Add lines
listen_addresses = 'localhost,10.8.1.2' # required for streaming replication
wal_level = replica
wal_log_hints = on
max_wal_senders = 3
wal_keep_size = 8
max_replication_slots = 2
synchronous_commit = remote_apply
synchronous_standby_names = 'lightningd'
full_page_writes = on

# Start PostgreSQL on NODE1 as primary
$ exit

$ sudo systemctl restart postgresql
~~~
## Configure NODE2 as Secondary
~~~
# Login NODE2
$ sudo -i -u postgres

# Prepare NODE2 to be secondary
$ touch /var/lib/postgresql/14/main/standby.signal
$ rm /var/lib/postgresql/14/main/postgresql.auto.conf

# Change configuration file
$ nano /etc/postgresql/14/main/postgresql.conf

# Comment out the following lines
#listen_addresses = 'localhost,10.8.1.4' # required for streaming replication
#wal_level = replica
#wal_log_hints = on
#max_wal_senders = 3
#wal_keep_size = 8
#max_replication_slots = 2
#synchronous_commit = remote_apply
#synchronous_standby_names = 'lightningd'
#full_page_writes = on

# Add these lines
primary_conninfo = 'host=10.8.1.2 port=5432 user=lightningusr password=''[PASSWORD]'' application_name=lightningd dbname=replication'
primary_slot_name = 'node_a_slot'


# Start PostgreSQL on NODE2 as secondary
$ exit
$ sudo systemctl enable postgresql
$ sudo systemctl start postgresql

~~~

## Check replication status on NODE1
~~~
# Login NODE1 (Now, it's primary node)
# Check Replication Status
$ sudo su - postgres -c 'psql -x -c "SELECT * from pg_stat_replication;"' 
$ sudo su - postgres -c 'psql -c "select usename, application_name, client_addr, state, sync_priority, sync_state from pg_stat_replication;"'

~~~

## Copy CLN Config Folder
~~~
# Copy file lightningd_data.tar to /tmp
$ cd /
$ sudo tar -xvf /tmp/lightningd_data.tar
~~~

## Start Core Lightning daemon
~~~
$ sudo systemctl enable lightningd.service
$ sudo systemctl start lightningd.service
$ sudo journalctl -f -u lightningd


~~~
