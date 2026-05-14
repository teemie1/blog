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
sudo pg_dropcluster 18 main --stop
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
sudo systemctl start postgresql@18-main
copy postgresql.conf ฿ pg_hba.conf

~~~
