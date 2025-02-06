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
$ docker compose up -d
~~~

## Configure nginx for nextcloud
~~~
sudo nano /etc/nginx/conf.d/nextcloud.conf
~~~
~~~
server {
    listen 80;
    listen [::]:80;
    root /var/www/html;
    server_name <your_domain>;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
        client_max_body_size 0;
    }

    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }

}
~~~
~~~
sudo nginx -t && sudo nginx -s reload
sudo certbot --nginx -d <domain> -m <email_address> --agree-tos
~~~


