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
# sudo pg_dropcluster 18 main --stop
# sudo pg_createcluster 18 main -- --no-data-checksums
sudo systemctl start postgresql@18-main
sudo systemctl stop postgresql@18-main
~~~

## Compatibility Check
~~~
sudo -iu postgres

/usr/lib/postgresql/18/bin/pg_upgrade \
  --old-datadir /var/lib/postgresql/14/main \
  --new-datadir /var/lib/postgresql/18/main \
  --old-bindir /usr/lib/postgresql/14/bin \
  --new-bindir /usr/lib/postgresql/18/bin \
  --check
~~~

## Performing upgrade
~~~
/usr/lib/postgresql/18/bin/pg_upgrade \
  --old-datadir /var/lib/postgresql/14/main \
  --new-datadir /var/lib/postgresql/18/main \
  --old-bindir /usr/lib/postgresql/14/bin \
  --new-bindir /usr/lib/postgresql/18/bin \
  --link
~~~

## Post upgrade task
~~~
copy postgresql.conf ฿ pg_hba.conf

~~~
