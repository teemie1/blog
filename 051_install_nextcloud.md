# Install NextCloud by docker container

## Create Directory & db.env file
~~~
$ mkdir nextcloud
$ cd nextcloud
$ sudo nano db.env
~~~
~~~
MYSQL_ROOT_PASSWORD=yoursqlrootpassword
MYSQL_PASSWORD=yourmysqlpassword
MYSQL_DATABASE=db
MYSQL_USER=yourmysqluser
~~~
## Create docker compose file
~~~
$ sudo nano docker-compose.yml
~~~
~~~
version: '3'

services:
  db:
    image: mariadb
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW --innodb-file-per-table=1 --skip-innodb-read-only-compressed
    volumes:
      - ./data/db:/var/lib/mysql
    env_file:
      - db.env

  redis:
    image: redis
    restart: always
    command: redis-server --requirepass your_redis_password

  app:
    image: nextcloud:latest
    restart: unless-stopped
    ports:
      - 8080:80
    links:
      - db
      - redis
    volumes:
      - ./data/nextcloud:/var/www/html
    environment:
      - MYSQL_HOST=db
      - REDIS_HOST_PASSWORD=your_redis_password
    env_file:
      - db.env
    depends_on:
      - db
      - redis
~~~
## Install Nextcloud on Docker
~~~
$ docker-compose up -d
~~~
