# Install Joplin - Note taking application server

~~~
$ sudo -i
$ mkdir joplin ; cd joplin
$ wget https://raw.githubusercontent.com/laurent22/joplin/dev/.env-sample
$ mv .env-sample .env
$ vi .env
APP_BASE_URL=http://10.7.0.1:22300
APP_PORT=22300
DB_CLIENT=pg
POSTGRES_PASSWORD=[PASSWORD]
POSTGRES_DATABASE=joplindb
POSTGRES_USER=joplinusr
POSTGRES_PORT=5432
POSTGRES_HOST=10.7.0.1

# Create db & user on only primary & standby server
$ sudo -i -u postgres
$ createuser --createdb --pwprompt --replication joplinusr # create user joplinusr (set a password)
$ createdb -O joplinusr joplindb # create database joplindb
$ exit

# Edit pg_hba.conf
$ vi /etc/postgresql/14/main/pg_hba.conf
$ systemctl restart postgresql

# Run docker container for joplin
$ docker run --restart always -d --name joplin --env-file .env --add-host=host.docker.internal:host-gateway -p 22300:22300 joplin/server:latest

# Verify log
$ docker logs joplin

# Check postgresql
$ sudo -iu postgres
$ psql -U joplinusr --host=localhost --port=5432 "dbname=joplindb"
\d
~~~
