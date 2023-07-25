# การติดตั้ง PostgreSQL และทำการ Replicate ข้อมูล

## ติดตั้ง PostgreSQL บนทั้งสองเครื่องในเน็ตเวิร์คเดียวกัน
 - primary node ip:   10.8.1.2
 - secondary node ip: 10.8.1.4
~~~
$ sudo apt install postgresql 
$ sudo -i -u postgres

$ psql
$ select version();

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




