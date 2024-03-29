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

## First Connect
~~~
Browse to http://[IP Address]:2283/
IOS/Androin App: http://[IP Address]:2283/api
~~~

## Configure Nginx
~~~

~~~

