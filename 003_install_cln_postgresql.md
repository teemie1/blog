# การติดตั้ง Core Lightning เพื่อใช้งานร่วมกับ PostgreSQL Database

NODE1: 10.7.0.1

NODE2: 10.7.0.11

## Install PostgreSQL on both machine via Ubuntu Repository
ติดตั้ง PostgreSQL 14.8 ลงบน NODE ทั้งสอง
~~~
# ติดตั้ง PostgreSQL
$ sudo apt install postgresql

# เช็ค PostgreSQL Version ที่ติดตั้งเรียบร้อย
$ sudo -i -u postgres
$ psql --host=10.7.0.2 --port=5433
Password for user postgres: 
psql (16.6 (Ubuntu 16.6-0ubuntu0.24.04.1), server 14.13 (Ubuntu 14.13-1.pgdg24.04+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.
~~~

## Change postgres password
~~~
$ sudo -i -u postgres
$ psql
postgres=# \password
# remember to take note of your password.
\q
~~~
## Check Status of PostgreSQL Running
~~~
$ ps -ef  | grep postgres
postgres   72314       1  0 11:06 ?        00:00:00 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
postgres   72316   72314  0 11:06 ?        00:00:00 postgres: 14/main: checkpointer
postgres   72317   72314  0 11:06 ?        00:00:00 postgres: 14/main: background writer
postgres   72318   72314  0 11:06 ?        00:00:00 postgres: 14/main: walwriter
postgres   72319   72314  0 11:06 ?        00:00:00 postgres: 14/main: autovacuum launcher
postgres   72320   72314  0 11:06 ?        00:00:00 postgres: 14/main: stats collector
postgres   72321   72314  0 11:06 ?        00:00:00 postgres: 14/main: logical replication launcher
tee        73378   69983  0 11:33 pts/0    00:00:00 grep --color=auto postgres

$ sudo systemctl status postgresql@14-main.service
● postgresql@14-main.service - PostgreSQL Cluster 14-main
     Loaded: loaded (/lib/systemd/system/postgresql@.service; enabled-runtime; vendor preset: enabled)
     Active: active (running) since Wed 2024-04-17 11:31:12 +07; 4 weeks 2 days ago
   Main PID: 4001186 (postgres)
      Tasks: 8 (limit: 28391)
     Memory: 126.5M
        CPU: 6h 26min 49.079s
     CGroup: /system.slice/system-postgresql.slice/postgresql@14-main.service
             ├─  89887 "postgres: 14/main: walsender lnd 10.7.0.11(52428) streaming 23/7130FD00" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             ├─4001186 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
             ├─4001188 "postgres: 14/main: checkpointer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             ├─4001189 "postgres: 14/main: background writer " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ">
             ├─4001190 "postgres: 14/main: walwriter " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             ├─4001191 "postgres: 14/main: autovacuum launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "">
             ├─4001192 "postgres: 14/main: stats collector " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" >
             └─4001193 "postgres: 14/main: logical replication launcher " "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "">

Apr 17 11:31:10 nuc11 systemd[1]: Starting PostgreSQL Cluster 14-main...
Apr 17 11:31:12 nuc11 systemd[1]: Started PostgreSQL Cluster 14-main.

$ sudo tail -f /var/log/postgresql/postgresql-14-main.log
2023-07-26 11:06:41.891 +07 [72314] LOG:  starting PostgreSQL 14.8 (Ubuntu 14.8-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, 64-bit
2023-07-26 11:06:41.891 +07 [72314] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2023-07-26 11:06:41.904 +07 [72314] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-07-26 11:06:41.940 +07 [72315] LOG:  database system was shut down at 2023-07-26 11:06:36 +07
2023-07-26 11:06:41.961 +07 [72314] LOG:  database system is ready to accept connections

# Ctrl-C to exit
~~~

## Create lightningd database and a user into PostgreSQL
~~~
# Create db & user on only primary & standby server
$ sudo -i -u postgres
$ createuser --createdb --pwprompt --replication lightningusr # create user lightningusr (set a password)
$ createdb -O lightningusr lightningdb # create database lightningdb
$ exit
~~~

## Download Core Lightning Software
~~~
# Check bitcoin.conf
$ sudo nano /data/bitcoin/bitcoin.conf
blocksonly=0
prune=n
bind=10.7.0.1

# Restart Bitcoin Core
$ sudo systemctl restart bitcoind

# Download Core Lightning Software and Checksum
$ cd /tmp
$ wget https://github.com/ElementsProject/lightning/releases/download/v24.08.2/clightning-v24.08.2-Ubuntu-28.04.tar.xz
$ wget https://github.com/ElementsProject/lightning/releases/download/v24.08.2/SHA256SUMS
$ wget https://github.com/ElementsProject/lightning/releases/download/v24.08.2/SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS
clightning-v24.08.2-Ubuntu-24.04.tar.xz: OK

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
$ sudo tar -xvf /tmp/clightning-v24.08.2-Ubuntu-24.04.tar.xz    # this will extract lightningd binary to the system

$ sudo mv /usr/lib/python3.12/EXTERNALLY-MANAGED /usr/lib/python3.12/EXTERNALLY-MANAGED.old

sudo -iu lightningd
pip install -r /usr/libexec/c-lightning/plugins/clnrest/requirements.txt
~~~
## Prepare environment for Core Lightning
~~~
$ sudo adduser --disabled-password --gecos "" lightningd
$ sudo usermod -a -G bitcoin,debian-tor lightningd
$ sudo adduser tee lightningd
$ sudo mkdir /data/lightningd
$ sudo mkdir /data/lightningd-plugins-available
$ sudo chown -R lightningd:lightningd /data/lightningd
$ sudo chown -R lightningd:lightningd /data/lightningd-plugins-available
$ sudo su - lightningd
$ ln -s /data/lightningd /home/lightningd/.lightning
$ ln -s /data/bitcoin /home/lightningd/.bitcoin
~~~
## Install prerequisite
~~~
exit
sudo apt install  python3-pip
apt install pkg-config
 mv /usr/lib/python3.12/EXTERNALLY-MANAGED /usr/lib/python3.12/EXTERNALLY-MANAGED.old
vi /usr/libexec/c-lightning/plugins/clnrest/requirements.txt
# Change coincurve>=18.0.0
sudo -iu lightningd
pip install coincurve
pip install -r /usr/libexec/c-lightning/plugins/clnrest/requirements.txt
~~~
## Configure CLN & PostgreSQL
~~~
$ sudo -i -u lightningd    # already created user for lightningd
$ cd /home/lightningd/.lightning
$ nano config
~~~
### Edit CLN Configuration File
~~~
# MiniBolt: cln configuration
# /home/lightningd/.lightning/config

alias=Satsdays.Com⚡ 
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
addr=statictor:127.0.0.1:9051/torport=9735
always-use-proxy=false

# CLEARNET
bind-addr=0.0.0.0:9735
announce-addr=[Public Ip]:9735

# Plugin
clnrest-port=3010
clnrest-protocol=https
clnrest-host=127.0.0.1

experimental-offers
experimental-splicing
experimental-dual-fund
experimental-anchors
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
~~~
$ sudo systemctl enable lightningd.service
$ sudo systemctl start lightningd.service
$ sudo journalctl -f -u lightningd
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_authid" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_subscription" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_database" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_db_role_setting" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_tablespace" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_auth_members" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_shdepend" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_shdescription" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_replication_origin" --- only superuser can vacuum it
Jul 26 13:45:41 nuc11 sh[42629]: WARNING:  skipping "pg_shseclabel" --- only superuser can vacuum it

~~~
## Allow user "tee" to work with CLN
~~~
$ ln -s /data/lightningd /home/tee/.lightning
$ sudo chmod -R g+x /data/lightningd/bitcoin/
$ logout
~~~
## Verify PostgreSQL DB
~~~
$ sudo -i -u postgres
$ psql -U lightningusr --host=localhost --port=5432 "dbname=lightningdb" 
lightningdb=> \d     # Check no table from lightning
                              List of relations
 Schema |                   Name                   |   Type   |    Owner
--------+------------------------------------------+----------+--------------
 public | blocks                                   | table    | lightningusr
 public | channel_blockheights                     | table    | lightningusr
 public | channel_configs                          | table    | lightningusr
 public | channel_configs_id_seq                   | sequence | lightningusr
 public | channel_feerates                         | table    | lightningusr
 public | channel_funding_inflights                | table    | lightningusr
...
lightningdb=> SELECT max(height) from blocks;
  max
--------
 800312
(1 row)

lightningdb=> \q
$ psql -U lightningusr --host=localhost --port=5432 "dbname=lightningdb" -t -c "SELECT max(height) from blocks;"      # Check blocks
 800312

$ exit
~~~
## Open port on firewall
~~~
$ sudo ufw allow 9735 comment 'allow CLN from outside'
~~~
Ref: [https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md](https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md)
