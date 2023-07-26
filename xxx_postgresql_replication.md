# การติดตั้ง PostgreSQL Replication สำหรับ Core Lightning

 - primary node ip:   10.8.1.2
 - secondary node ip: 10.8.1.4


## Allow connections from the standby server to the primary
~~~
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
host     replication     lightningusr     10.8.1.4/32 md5
host     postgres        postgres         10.8.1.4/32 md5 # for dyagnostic purposes
sudo systemctl restart postgresql
~~~
## Allow connections from the primary server to the standby
~~~
$ sudo nano /etc/postgresql/14/main/pg_hba.conf
host     replication     lightningusr     10.8.1.2/32 md5
host     postgres        postgres         10.8.1.2/32 md5 # for dyagnostic purposes
sudo systemctl restart postgresql
~~~


## แก้ไข password ของ PostgreSQL
~~~
$ sudo -i -u postgres
# psql
# postgres=# \password
# remember to take note of your password.
# \q
~~~

##
~~~

~~~

##
~~~

~~~




