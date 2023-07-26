# การติดตั้ง Core Lightning เพื่อใช้งานร่วมกับ PostgreSQL Database

NODE1: 10.8.1.2

NODE2: 10.8.1.4

## Install PostgresQL on both machine via Ubuntu Repository
ติดตั้ง PostgreSQL 14.0 ลงบน NODE ทั้งสอง
~~~
# ติดตั้ง PostgreSQL
$ sudo apt install postgresql

# เช็ค PostgreSQL Version ที่ติดตั้งเรียบร้อย
$ sudo -i -u postgres
$ psql
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# select version();
                                                                version
----------------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.8 (Ubuntu 14.8-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, 64-bit
(1 row)

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

$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Wed 2023-07-26 11:06:44 +07; 26min ago
    Process: 72332 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 72332 (code=exited, status=0/SUCCESS)
        CPU: 2ms

Jul 26 11:06:44 bitwarden systemd[1]: Starting PostgreSQL RDBMS...
Jul 26 11:06:44 bitwarden systemd[1]: Finished PostgreSQL RDBMS.

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

# Restart Bitcoin Core
$ sudo systemctl restart bitcoind

# Download Core Lightning Software and Checksum
$ cd /tmp
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.05.2/clightning-v23.05.2-Ubuntu-22.04.tar.xz
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.05.2/SHA256SUMS
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.05.2/SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS
clightning-v23.05.2-Ubuntu-22.04.tar.xz: OK

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
$ sudo tar -xvf /tmp/clightning-v23.05.2-Ubuntu-22.04.tar.xz    # this will extract lightningd binary to the system
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

alias=teemie⚡ 
rgb=FFA500
network=bitcoin
log-file=/data/lightningd/cln.log
log-level=info
# for admin to interact with lightning-cli
rpc-file-mode=0660

# default fees and channel min size
fee-base=1000
fee-per-satoshi=1
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
announce-addr=[Public Ip]:9736

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
$ sudo ufw allow 9736 comment 'allow CLN from outside'
~~~
