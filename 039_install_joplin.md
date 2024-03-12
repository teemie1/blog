# Install Joplin - Note taking application server

~~~
$ sudo -i
$ mkdir joplin ; cd joplin
$ wget https://raw.githubusercontent.com/laurent22/joplin/dev/.env-sample
$ mv .env-sample .env
$ vi .env
DB_CLIENT=pg
POSTGRES_CONNECTION_STRING=postgresql://joplin:joplin@localhost:5432/joplin

$ docker run --env-file .env -p 22300:22300 joplin/server:latest

~~~
