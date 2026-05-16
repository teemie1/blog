# Upgrade PostgreSQL version 14 to 18

## Backup PostgreSQL v14
~~~
pg_basebackup -h [host] -p 5432 -U [user] -D /path/to/backup/dir -Fp -Xs -P -v
pg_basebackup -h localhost -p 5433 -U lightningusr -D /tmp/lightningdb.tar  -Ft -Xs -P -v --password
~~~

## Stop PostgreSQL
~~~
sudo systemctl stop postgresql@14-main
~~~

## Install PostgreSQL v18
~~~
sudo apt-get install postgresql-18
sudo pg_lsclusters
sudo pg_dropcluster 18 main
# sudo pg_createcluster 18 main -- --no-data-checksums
# sudo pg_ctlcluster 18 main start
sudo pg_dropcluster 18 main --stop

sudo touch /var/lib/postgresql/14/main/postgresql.auto.conf
sudo chown postgres:postgres /var/lib/postgresql/14/main/postgresql.auto.conf
~~~

## Compatibility Check
~~~
sudo -iu postgres
/usr/lib/postgresql/18/bin/pg_upgrade \
  --old-datadir /var/lib/postgresql/14/main \
  --new-datadir /var/lib/postgresql/18/main \
  --old-bindir /usr/lib/postgresql/14/bin \
  --new-bindir /usr/lib/postgresql/18/bin \
  -o "-c config_file=/etc/postgresql/14/main/postgresql.conf" \
  -O "-c config_file=/etc/postgresql/18/main/postgresql.conf" \
  --check
~~~

## Performing upgrade
~~~
sudo pg_upgradecluster -m upgrade 14 main -v 18

or

sudo -iu postgres
/usr/lib/postgresql/18/bin/pg_upgrade \
  --old-datadir /var/lib/postgresql/14/main \
  --new-datadir /var/lib/postgresql/18/main \
  --old-bindir /usr/lib/postgresql/14/bin \
  --new-bindir /usr/lib/postgresql/18/bin \
  -o "-c config_file=/etc/postgresql/14/main/postgresql.conf" \
  -O "-c config_file=/etc/postgresql/18/main/postgresql.conf" \
  --link
~~~

## Post upgrade task
~~~
# Start postgresql v18
sudo pg_ctlcluster 18 main start
or
sudo systemctl start postgresql@18-main

# Verify
psql -U lightningusr --host=localhost --port=5433 "dbname=lightningdb" -t -c "SELECT max(height) from blocks;"
~~~

# Upgrade Log
~~~
root@vm-psql01:~# sudo pg_upgradecluster -m upgrade 14 main
Upgrading cluster 14/main to 18/main ...
Restarting old cluster with restricted connections...
Notice: extra pg_ctl/postgres options given, bypassing systemctl for start operation
Stopping old cluster...
Creating new PostgreSQL cluster 18/main ...
/usr/lib/postgresql/18/bin/initdb -D /var/lib/postgresql/18/main --auth-local peer --auth-host scram-sha-256 --no-instructions --encoding UTF8 --lc-collate en_US.UTF-8 --lc-ctype en_US.UTF-8 --no-data-checksums
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with this locale configuration:
  locale provider:   libc
  LC_COLLATE:  en_US.UTF-8
  LC_CTYPE:    en_US.UTF-8
  LC_MESSAGES: C.UTF-8
  LC_MONETARY: C.UTF-8
  LC_NUMERIC:  C.UTF-8
  LC_TIME:     C.UTF-8
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/18/main ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default "max_connections" ... 100
selecting default "shared_buffers" ... 128MB
selecting default time zone ... Etc/UTC
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

Copying old configuration files...
Copying old start.conf...
Copying old pg_ctl.conf...
Running init phase upgrade hook scripts ...

/usr/lib/postgresql/18/bin/pg_upgrade -b /usr/lib/postgresql/14/bin -B /usr/lib/postgresql/18/bin -p 5433 -P 5432 -d /etc/postgresql/14/main -D /etc/postgresql/18/main
Finding the real data directory for the source cluster        ok
Finding the real data directory for the target cluster        ok
Performing Consistency Checks
-----------------------------
Checking cluster versions                                     ok
Checking database connection settings                         ok
Checking database user is the install user                    ok
Checking for prepared transactions                            ok
Checking for contrib/isn with bigint-passing mismatch         ok
Checking data type usage                                      ok
Checking for not-null constraint inconsistencies              ok
Creating dump of global objects                               ok
Creating dump of database schemas                             
                                                              ok
Checking for presence of required libraries                   ok
Checking database user is the install user                    ok
Checking for prepared transactions                            ok
Checking for new cluster tablespace directories               ok

If pg_upgrade fails after this point, you must re-initdb the
new cluster before continuing.

Performing Upgrade
------------------
Setting locale and encoding for new cluster                   ok
Analyzing all rows in the new cluster                         ok
Freezing all rows in the new cluster                          ok
Deleting files from new pg_xact                               ok
Copying old pg_xact to new server                             ok
Setting oldest XID for new cluster                            ok
Setting next transaction ID and epoch for new cluster         ok
Deleting files from new pg_multixact/offsets                  ok
Copying old pg_multixact/offsets to new server                ok
Deleting files from new pg_multixact/members                  ok
Copying old pg_multixact/members to new server                ok
Setting next multixact ID and offset for new cluster          ok
Resetting WAL archives                                        ok
Setting the default char signedness for new cluster           ok
Setting frozenxid and minmxid counters in new cluster         ok
Restoring global objects in the new cluster                   ok
Restoring database schemas in the new cluster                 
                                                              ok
Copying user relation files                                   
                                                              ok
Setting next OID for new cluster                              ok
Sync data directory to disk                                   ok
Creating script to delete old cluster                         ok
Checking for extension updates                                ok

Upgrade Complete
----------------
Some statistics are not transferred by pg_upgrade.
Once you start the new server, consider running these two commands:
    /usr/lib/postgresql/18/bin/vacuumdb --all --analyze-in-stages --missing-stats-only
    /usr/lib/postgresql/18/bin/vacuumdb --all --analyze-only
Running this script will delete the old cluster's data files:
    ./delete_old_cluster.sh
pg_upgradecluster: pg_upgrade output scripts are in /var/log/postgresql/pg_upgradecluster-14-18-main.VLs_
Disabling automatic startup of old cluster...
Starting upgraded cluster on port 5433...
Running finish phase upgrade hook scripts ...
vacuumdb: processing database "joplindb": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "lightningdb": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "lightningdb2": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "lnbitsdb": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "lndb": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "postgres": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "template1": Generating minimal optimizer statistics (1 target)
vacuumdb: processing database "joplindb": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "lightningdb": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "lightningdb2": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "lnbitsdb": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "lndb": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "postgres": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "template1": Generating medium optimizer statistics (10 targets)
vacuumdb: processing database "joplindb": Generating default (full) optimizer statistics
vacuumdb: processing database "lightningdb": Generating default (full) optimizer statistics
vacuumdb: processing database "lightningdb2": Generating default (full) optimizer statistics
vacuumdb: processing database "lnbitsdb": Generating default (full) optimizer statistics
vacuumdb: processing database "lndb": Generating default (full) optimizer statistics
vacuumdb: processing database "postgres": Generating default (full) optimizer statistics
vacuumdb: processing database "template1": Generating default (full) optimizer statistics
vacuumdb: vacuuming database "joplindb"
vacuumdb: vacuuming database "lightningdb"
vacuumdb: vacuuming database "lightningdb2"
vacuumdb: vacuuming database "lnbitsdb"
vacuumdb: vacuuming database "lndb"
vacuumdb: vacuuming database "postgres"
vacuumdb: vacuuming database "template1"

Success. Please check that the upgraded cluster works. If it does,
you can remove the old cluster with
    pg_dropcluster 14 main

Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 down   postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
Ver Cluster Port Status Owner    Data directory              Log file
18  main    5433 online postgres /var/lib/postgresql/18/main /var/log/postgresql/postgresql-18-main.log
root@vm-psql01:~#
~~~
