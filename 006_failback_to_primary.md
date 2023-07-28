# Scenario 2: Failback from NODE2 to NODE1
 - NODE1: 10.8.1.2, 10.8.0.2
 - NODE2: 10.8.1.4, 10.8.0.4
 - VPS: 165.232.161.68, 10.8.0.1

## Stop Core Lightning on NODE2
~~~
# Login NODE2
$ sudo -i -u lightningd
$ lightning-cli stop
$ exit
$ sudo systemctl stop postgresql
$ sudo systemctl disable postgresql
~~~

## Promote PostgreSQL on NODE1 as primary
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

## Configure PostgreSQL on NODE2 as secondary and start replication
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

## Start Core Lighting on NODE1
~~~
# Login NODE1
$ sudo systemctl enable lightningd
$ sudo systemctl start lightningd
~~~

## Checking and verification
~~~
# Login NODE1
$ sudo -i -u lightningd
$ lightning-cli getinfo
~~~



Ref: https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md

