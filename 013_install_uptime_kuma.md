# Install Uptime Kuma for Monitoring my node.

## Install Uptime Kuma via docker
~~~
$ sudo docker run -d --restart=always -p 3001:3001 -v uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
# Check docker container is running
$ sudo docker ps -a
~~~

## Go to Uptime Kuma
~~~
# Browse to http://[IP Address of Server]:3001
# Default User: admin
# Default Password: admin
~~~

## 
~~~

~~~

