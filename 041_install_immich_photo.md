# Install Immich - Photo & Video Management Solution

## Install Immich
~~~
sudo -i
mkdir ./immich-app
cd ./immich-app
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
docker compose up -d
~~~

## Configure Nginx & Certificate
~~~
$ sudo nano /etc/nginx/sites-available/immich.conf

server {
    server_name immich.satsdays.com;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:2283;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}

$ sudo ln -s /etc/nginx/sites-available/immich.conf /etc/nginx/sites-enabled/immich.conf

# Restart nginx
$ sudo nginx -t
$ sudo systemctl start nginx

# Map DNS A record to IP of VM machine (see DNS Settings below)

# Request SSL cert from letsencrypt/certbot
$ sudo certbot --nginx -d immich.satsdays.com
~~~

## First Connect
~~~
Browse to https://[Domain Name]/
IOS/Androin App: https://[Domain Name]/api
~~~
