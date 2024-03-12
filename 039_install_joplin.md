# Install Joplin - Note taking application server

~~~
$ sudo -i
$ mkdir joplin ; cd joplin
$ wget https://raw.githubusercontent.com/laurent22/joplin/dev/.env-sample
$ mv .env-sample .env
$ vi .env
DB_CLIENT=pg
POSTGRES_CONNECTION_STRING=postgresql://joplinusr:[PASSWORD]@localhost:5432/joplindb

# Create db & user on only primary & standby server
$ sudo -i -u postgres
$ createuser --createdb --pwprompt --replication joplinusr # create user joplinusr (set a password)
$ createdb -O joplinusr joplindb # create database joplindb
$ exit

$ docker run --env-file .env -p 22300:22300 joplin/server:latest

~~~
