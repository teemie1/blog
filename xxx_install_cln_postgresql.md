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

##
~~~
~~~



