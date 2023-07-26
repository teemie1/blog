# การติดตั้ง Core Lightning เพื่อใช้งานร่วมกับ PostgreSQL Database

NODE1: 10.8.1.2
NODE2: 10.8.1.4

## Install PostgresQL on both machine via Ubuntu Repository
ติดตั้ง PostgreSQL 14.0 ลงบน NODE ทั้งสอง
~~~
# ติดตั้ง PostgreSQL
$ sudo apt install postgresql

# เช็ค PostgreSQL Version ที่ติดตั้งเรียบร้อย
$ sudo -i -u postgres
$ psql
psql (14.8 (Ubuntu 14.8-0ubuntu0.22.04.1))
Type "help" for help.

postgres=# select version();
                                                                version
----------------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 14.8 (Ubuntu 14.8-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, 64-bit
(1 row)

~~~

## Change postgres password
~~~
$ sudo -i -u postgres
$ psql
postgres=# \password
# remember to take note of your password.
\q
~~~
## Check Status of PostgreSQL Running
~~~
$ ps -ef  | grep postgres
postgres   72314       1  0 11:06 ?        00:00:00 /usr/lib/postgresql/14/bin/postgres -D /var/lib/postgresql/14/main -c config_file=/etc/postgresql/14/main/postgresql.conf
postgres   72316   72314  0 11:06 ?        00:00:00 postgres: 14/main: checkpointer
postgres   72317   72314  0 11:06 ?        00:00:00 postgres: 14/main: background writer
postgres   72318   72314  0 11:06 ?        00:00:00 postgres: 14/main: walwriter
postgres   72319   72314  0 11:06 ?        00:00:00 postgres: 14/main: autovacuum launcher
postgres   72320   72314  0 11:06 ?        00:00:00 postgres: 14/main: stats collector
postgres   72321   72314  0 11:06 ?        00:00:00 postgres: 14/main: logical replication launcher
tee        73378   69983  0 11:33 pts/0    00:00:00 grep --color=auto postgres

$ sudo systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Wed 2023-07-26 11:06:44 +07; 26min ago
    Process: 72332 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 72332 (code=exited, status=0/SUCCESS)
        CPU: 2ms

Jul 26 11:06:44 bitwarden systemd[1]: Starting PostgreSQL RDBMS...
Jul 26 11:06:44 bitwarden systemd[1]: Finished PostgreSQL RDBMS.

$ sudo tail -f /var/log/postgresql/postgresql-14-main.log
2023-07-26 11:06:41.891 +07 [72314] LOG:  starting PostgreSQL 14.8 (Ubuntu 14.8-0ubuntu0.22.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, 64-bit
2023-07-26 11:06:41.891 +07 [72314] LOG:  listening on IPv4 address "127.0.0.1", port 5432
2023-07-26 11:06:41.904 +07 [72314] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-07-26 11:06:41.940 +07 [72315] LOG:  database system was shut down at 2023-07-26 11:06:36 +07
2023-07-26 11:06:41.961 +07 [72314] LOG:  database system is ready to accept connections

# Ctrl-C to exit
~~~

## 
~~~

~~~

