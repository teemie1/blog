# Upgrade PostgreSQL version 14 to 18

## Install PostgreSQL v18
~~~
sudo apt-get install postgresql-18
~~~

## Stop PostgreSQL
~~~
sudo systemctl stop postgresql@14-main
sudo systemctl stop postgresql@18-main
~~~

## Compatibility Check
~~~
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
