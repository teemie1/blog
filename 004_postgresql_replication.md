# การติดตั้ง PostgreSQL Replication สำหรับ Core Lightning

 - primary node ip:   192.168.1.57
 - secondary node ip: 192.168.1.106


## Allow connections from the standby server to the primary
~~~
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
# Add following lines
host    replication     lightningusr    192.168.1.106/32             md5
host    postgres        postgres        192.168.1.106/32             md5 # for dyagnostic purposes
$ sudo systemctl restart postgresql
~~~
## Allow connections from the primary server to the standby
~~~
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
# Add following lines
host    replication     lightningusr    192.168.1.57/32             md5
host    postgres        postgres        192.168.1.57/32             md5 # for dyagnostic purposes
$ sudo systemctl restart postgresql
~~~


## Create a replication slot:
~~~
$ sudo -i -u postgres 
$ psql --host=localhost --port=5433
postgres=# SELECT * FROM pg_create_physical_replication_slot('node_a_slot');
slot_name | lsn
-------------+-----
node_a_slot |

postgres=# SELECT slot_name, slot_type, active FROM pg_replication_slots;
slot_name | slot_type | active
-------------+-----------+--------
node_a_slot | physical | f
(1 row)
postgres= \q
$ exit
~~~

## Open the postgresql.conf file on primary and set the following configuration.
~~~
$ sudo locale-gen en_US.UTF-8
$ sudo vi /etc/postgresql/14/main/postgresql.conf
# Change port=5433

listen_addresses = 'localhost,192.168.1.106' # required for streaming replication
wal_level = replica
wal_log_hints = on
max_wal_senders = 3
wal_keep_size = 8
max_replication_slots = 2
synchronous_commit = remote_apply
synchronous_standby_names = 'lightningd' # gbd Mar 28 Gen 2020 16:13:46 CET
full_page_writes = on
$ sudo systemctl restart postgresql
~~~

## Setting Up the Standby server for streaming replication
~~~
$ sudo locale-gen en_US.UTF-8
$ sudo systemctl stop postgresql
$ sudo mv /var/lib/postgresql/14/main/ /var/lib/postgresql/14/main.backup
$ sudo -i -u postgres
$ pg_basebackup -h 192.168.1.57 -p 5433 -U lightningusr -D /var/lib/postgresql/14/main/ -P --password --slot node_a_slot
# enter the password for user lightningusr when prompted

$ touch /var/lib/postgresql/14/main/standby.signal

$ vi /etc/postgresql/14/main/postgresql.conf
# Change port=5433

primary_conninfo = 'host=192.168.1.57 port=5433 user=lightningusr password=''[PASSWORD]'' application_name=lightningd dbname=replication'
primary_slot_name = 'node_a_slot'
#hot_standby = on

# Remove auto conf file /var/lib/postgresql/14/main/postgresql.auto.conf
$ rm /var/lib/postgresql/14/main/postgresql.auto.conf
~~~

## Start postgresql on both servers
~~~
$ sudo systemctl start postgresql
~~~
## Check PostgreSQL Log
~~~
$ tail /var/log/postgresql/postgresql-14-main.log
~~~
## Check replication at primary server
~~~
# Check Replication Status
$ sudo su - postgres -c 'psql --host localhost --port 5433 -x -c "SELECT * from pg_stat_replication;"' 
$ sudo su - postgres -c 'psql --host localhost --port 5433 -c "select usename, application_name, client_addr, state, sync_priority, sync_state from pg_stat_replication;"'

~~~
## Check block height at standby server
~~~
$ sudo -i -u postgres
$ psql -U lightningusr --host=localhost --port=5433 "dbname=lightningdb" -t -c "SELECT max(height) from blocks;"
Password for user lightningusr:
 800337

~~~
Ref: [https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md](https://github.com/gabridome/docs/blob/master/c-lightning_with_postgresql_reliability.md)
