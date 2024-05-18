# Scenario 1: Failover from NODE1 to NODE2

 - NODE1: 10.7.0.1, 10.8.0.2
 - NODE2: 10.7.0.11, 10.8.0.4
 - VPS:  165.232.161.68, 10.8.0.1

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
# Comment out
#primary_conninfo = 'host=10.7.0.1 port=5432 user=lightningusr password=''[PASSWORD]'' application_name=lightningd dbname=replication'
#primary_slot_name = 'node_a_slot'
# Add lines
listen_addresses = 'localhost,10.7.0.11' # required for streaming replication
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

$ sudo systemctl restart postgresql
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
#listen_addresses = 'localhost,10.8.1.2' # required for streaming replication
#wal_level = replica
#wal_log_hints = on
#max_wal_senders = 3
#wal_keep_size = 8
#max_replication_slots = 2
#synchronous_commit = remote_apply
#synchronous_standby_names = 'lightningd'
#full_page_writes = on

# Add these lines
primary_conninfo = 'host=10.7.0.11 port=5432 user=lightningusr password=''[PASSWORD]'' application_name=lightningd dbname=replication'
primary_slot_name = 'node_a_slot'


# Start PostgreSQL on NODE1 as secondary
$ exit
$ sudo systemctl enable postgresql
$ sudo systemctl start postgresql

~~~
## Check replication status on NODE2
~~~
# Login NODE2 (Now, it's primary node)
# Check Replication Status
$ sudo su - postgres -c 'psql -x -c "SELECT * from pg_stat_replication;"' 
$ sudo su - postgres -c 'psql -c "select usename, application_name, client_addr, state, sync_priority, sync_state from pg_stat_replication;"'

~~~
## Copy config file from NODE1 to NODE2
~~~
# Login NODE1
$ sudo tar -cvf /tmp/cln_config.tar /data/lightningd*
$ sudo chown tee.tee /tmp/cln_config.tar
$ scp /tmp/cln_config.tar 10.8.1.4:/tmp

# Login NODE2
$ cd /
$ sudo rm -rf /data/lightningd*
$ sudo tar -xvf /tmp/cln_config.tar
$ sudo -i -u lightningd
$ ln -s /data/lightningd .lightning
$ rm /data/lightningd/config
$ rm /data/lightningd/cln.log
$ /usr/bin/lightning-hsmtool encrypt /data/lightningd/bitcoin/hsm_secret
# Enter the password
Successfully encrypted hsm_secret. You'll now have to pass the --encrypted-hsm startup option.

# Delete tar file
$ exit
$ sudo rm /tmp/cln_config.tar

# Open port at fw
$ sudo ufw allow 9737/tcp comment 'allow cln(secondary) from anywhere'
~~~

## Start Core Lighting on NODE2
~~~
# Login NODE2
$ sudo -i -u lightningd
$ /usr/bin/lightningd  --alias=teemie⚡ \
                       --rgb=FFA500 \
                       --bitcoin-rpcuser=umbrel \
                       --bitcoin-rpcpassword='[PASSWORD of Bitcoind on Umbrel]' \
                       --bitcoin-rpcport=8332 \
                       --bitcoin-rpcconnect=10.8.0.205 \
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
                       --bind-addr=0.0.0.0:9737 \
                       --announce-addr=165.232.161.68:9737 \
                       --always-use-proxy=false \
                       --encrypted-hsm \
                       --daemon 

~~~

## Checking and verification
~~~
# Login NODE2 (Can't use admin/tee user on NODE2)
$ sudo -i -u lightningd
$ lightning-cli getinfo
~~~

## Change forward port on VPS
~~~
# Login to VPS Clearnet
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 9737 -j DNAT --to-destination 10.8.0.4
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 9737 -d 10.8.0.4 -j SNAT --to-source 10.8.0.1

$ sudo netfilter-persistent save
$ sudo systemctl enable netfilter-persistent
~~~

Ref: https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md
