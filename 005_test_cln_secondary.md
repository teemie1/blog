# Scenario 1: Fail over from NODE1 to NODE2

NODE1: 10.8.1.2
NODE2: 10.8.1.4

## Stop Core Lightning on NODE1
~~~
# Login NODE1
$ sudo systemctl stop lightningd
$ sudo systemctl stop postgresql
$ sudo systemctl disable lightningd
$ sudo systemctl disable postgresql
~~~

## Promote PostgreSQL on NODE2 as primary
~~~
# Login NODE2
$ sudo -i -u postgres

# Check cluster version & name
$ pg_lsclusters

# Check cluster configuration
$ pg_config

# In case for fast failover
$ pg_ctlcluster 14 main promote

# In case for prepare replication back to NODE1
$ rm /var/lib/postgresql/14/main/standby.signal
# Change configuration file
$ nano /etc/postgresql/14/main/postgresql.conf
listen_addresses = 'localhost,10.8.1.4' # required for streaming replication
wal_level = replica
wal_log_hints = on
max_wal_senders = 3
wal_keep_size = 8
max_replication_slots = 2
synchronous_commit = remote_apply
synchronous_standby_names = 'lightningd'
full_page_writes = on

# Start PostgreSQL on NODE2 as primary
$ exit

$ sudo systemctl start postgresql
~~~

## Configure PostgreSQL on NODE1 as secondary and start replication
~~~
# Login NODE1
$ sudo -i -u postgres

# Prepare NODE1 to be secondary
$ touch /var/lib/postgresql/14/main/standby.signal
$ rm /var/lib/postgresql/14/main/postgresql.auto.conf

# Change configuration file
$ nano /etc/postgresql/14/main/postgresql.conf

# Comment out the following lines
#listen_addresses = 'localhost,10.8.0.4' # required for streaming replication
#wal_level = replica
#wal_log_hints = on
#max_wal_senders = 3
#wal_keep_size = 8
#max_replication_slots = 2
#synchronous_commit = remote_apply
#synchronous_standby_names = 'lightningd'
#full_page_writes = on

# Add these lines
primary_conninfo = 'host=10.8.1.4 port=5432 user=lightningusr password=''[PASSWORD]'' application_name=lightningd dbname=replication'
primary_slot_name = 'node_a_slot'


# Start PostgreSQL on NODE2 as primary
$ exit
$ sudo systemctl start postgresql

~~~

## Start Core Lighting on NODE2
~~~
Login NODE2
$ sudo -i -u lightningd
$ /usr/bin/lightningd  --alias=teemie⚡ \
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
~~~

## Checking and verification
~~~

~~~
~~~

#             --wallet=postgres://lightningusr:ch!chaK0rn9103@localhost:5432/lightningdb \
~~~
Ref: https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md
