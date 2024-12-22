# Command for PostgreSQL

## Delete Database
~~~
sudo -iu postgres
psql -U postgres
DROP DATABASE lightningdb;
~~~

## Create User & Database
~~~
$ sudo -i -u postgres
$ createuser --createdb --pwprompt --replication lightningusr # create user lightningusr (set a password)
$ createdb -O lightningusr lightningdb # create database lightningdb
~~~

## Connect to Database and list table
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
## Change owner of database
~~~
ALTER DATABASE lnbitsdb OWNER TO lnbits;
~~~
## Delete user
~~~
DROP USER lnbitsusr;
~~~
