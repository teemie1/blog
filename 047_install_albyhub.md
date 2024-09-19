# Install Alby Hub

~~~
mkdir alby-hub
cd alby-hub
nano docker-compose.yml
~~~

~~~
services:
  albyhub:
    platform: linux/amd64
    container_name: albyhub
    image: ghcr.io/getalby/hub:latest
    volumes:
      - ./albyhub-data:/data
    ports:
      - "8080:8080"
    environment:
      - WORK_DIR=/data/albyhub
      - LOG_EVENTS=true
~~~

~~~
sudo docker compose up -d --build
sudo ufw allow 8080/tcp comment 'Allow alby hub'
~~~
Browse to https://[IP Address]:8080
