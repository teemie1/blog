# Install Joplin - Note taking application server

~~~
$ sudo -i
$ mkdir joplin ; cd joplin
$ wget https://raw.githubusercontent.com/laurent22/joplin/dev/.env-sample
$ mv .env-sample .env
$ vi .env
APP_BASE_URL=https://joplin.satsdays.com
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
$ docker run --restart always -d --name joplin --env-file .env --add-host=host.docker.internal:host-gateway -p 127.0.0.1:22300:22300 joplin/server:latest

# Verify log
$ docker logs joplin

# Check postgresql
$ sudo -iu postgres
$ psql -U joplinusr --host=localhost --port=5432 "dbname=joplindb"
\d
\q
$ exit

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

# Open browser https://joplin.satsdays.com

# Default admin user
Username: admin@localhost
Password: admin
~~~
