# Install ZABBIX with PostgreSQL Database


## Install Ubuntu Linux Server
Install Ubuntu Server 22.04.2
~~~
$ sudo apt update
$ sudo apt upgrade
~~~

## Install PostgreSQL Database
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

# change postgres password
postgres=# \password
# remember to take note of your password.
\q
$ exit
~~~

## Install and configure Zabbix for your platform
###  Install Zabbix repository
~~~
$ cd /tmp
$ wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
$ sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
$ sudo apt update
~~~
### Install Zabbix server, frontend, agent
~~~
$ sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
~~~
### Create initial database
~~~
$ sudo -u postgres createuser --pwprompt zabbix
$ sudo -u postgres createdb -O zabbix zabbix
$ zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
~~~
###  Configure the database for Zabbix server
~~~
# Edit file /etc/zabbix/zabbix_server.conf
$ sudo nano /etc/zabbix/zabbix_server.conf
DBPassword=password
~~~
### Configure PHP for Zabbix frontend
~~~
# Edit file /etc/zabbix/nginx.conf uncomment and set 'listen' and 'server_name' directives.
$ sudo nano /etc/zabbix/nginx.conf
listen 8080;
server_name example.com;
~~~
### Start Zabbix server and agent processes
~~~
$ sudo systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm
$ sudo systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm
~~~
### Open Zabbix UI web page
~~~
# Open browser and go to URL: http://[IP Address]:8080
# User: Admin
# password: zabbix 
~~~
