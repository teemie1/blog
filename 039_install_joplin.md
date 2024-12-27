# Install Joplin - Note taking application server

~~~
$ sudo -i
$ mkdir joplin ; cd joplin
$ sudo nano docker-compose.yml

version: '3'

services:
    db:
        image: postgres:16
        container_name: joplin-db
        volumes:
            - ./data/postgres:/var/lib/postgresql/data
        ports:
            - "15432:5432"
        restart: unless-stopped
        environment:
            - POSTGRES_PASSWORD=password
            - POSTGRES_USER=joplin-user
            - POSTGRES_DB=joplindb
    app:
        image: joplin/server:latest
        container_name: joplin-app
        depends_on:
            - db
        ports:
            - "22300:22300"
        restart: unless-stopped
        environment:
            - APP_PORT=22300
            - APP_BASE_URL=https://joplin.satsdays.com
            - DB_CLIENT=pg
            - POSTGRES_PASSWORD=password
            - POSTGRES_DATABASE=joplindb
            - POSTGRES_USER=joplin-user
            - POSTGRES_PORT=5432
            - POSTGRES_HOST=db

$ sudo docker compose -f docker-compose.yml up -d
$ docker logs joplin-app
~~~


## Configure Nginx & Certificate
~~~
$ sudo nano /etc/nginx/sites-available/joplin.conf

server {
    server_name joplin.satsdays.com;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:22300;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

$ sudo ln -s /etc/nginx/sites-available/joplin.conf /etc/nginx/sites-enabled/joplin.conf

# Restart nginx
$ sudo nginx -t
$ sudo systemctl start nginx

# Map DNS A record to IP of VM machine (see DNS Settings below)

# Request SSL cert from letsencrypt/certbot
$ sudo certbot --nginx -d joplin.satsdays.com
~~~

## Open browser https://joplin.satsdays.com
~~~
# Default admin user
Username: admin@localhost
Password: admin
~~~
