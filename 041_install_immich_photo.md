# Install Immich - Photo & Video Management Solution


~~~
sudo -i
mkdir ./immich-app
cd ./immich-app
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
docker compose up -d
~~~
