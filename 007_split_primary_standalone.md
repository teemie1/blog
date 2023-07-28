# Scenario 3: Split PostgreSQL Replication and NODE1 as standalone

 - NODE1: 10.8.1.2, 10.8.0.2
 - NODE2: 10.8.1.4, 10.8.0.4

## Stop PostgreSQL on NODE1
~~~
# Login NODE1
$ sudo systemctl stop postgresql
~~~

## Change PostgreSQL config file
~~~
$ sudo -i -u postgres
$ nano /etc/postgresql/14/main/postgresql.conf
# Comment out lines. Let only listen_addresses
listen_addresses = 'localhost,10.8.1.4' 
#wal_level = replica
#wal_log_hints = on
#max_wal_senders = 3
#wal_keep_size = 8
#max_replication_slots = 2
#synchronous_commit = remote_apply
#synchronous_standby_names = 'lightningd'
#full_page_writes = on

# Start PostgreSQL on NODE1 as standalone
$ exit
$ sudo systemctl start postgresql
~~~

## Check Core Lightning
~~~
$ lightning-cli getinfo
~~~

