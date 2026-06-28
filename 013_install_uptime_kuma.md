# Install Uptime Kuma for Monitoring my node.

## Install Uptime Kuma via docker
~~~
$ mkdir /root/uptime-kuma
$ cd /root/uptime-kuma
$ vi docker-compose.yml

version: "3"
services:
   uptime-kuma:
      container_name: uptime-kuma
      image: louislam/uptime-kuma:2
      volumes:
         - ./data:/app/data
      ports:
         - 3001:3001
      restart: unless-stopped

$ docker compose up -d
# Check docker container is running
$ sudo docker ps -a
~~~


## Go to Uptime Kuma
~~~
# Browse to http://[IP Address of Server]:3001
# Select DB Type and set username/password
~~~




