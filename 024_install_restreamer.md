# Install Restreamer and replace to NGINX-RTMP 

## Disable NGINX-RTMP
~~~
# Disable RTMP
$ sudo cp /etc/nginx/nginx.conf /tmp/nginx.conf.rtmp
$ sudo vi /etc/nginx/nginx.conf
# Remove RTMP Application from nginx.conf

# Disable port 8080
$ sudo cp /etc/nginx/sites-available/rtmp /tmp/rtmp.8080
$ sudo vi /etc/nginx/sites-available/rtmp
# Remove Server listen 8080 from rtmp

# Restart Nginx
$ sudo systemctl reload nginx

# Check port open
$ sudo ss -ntap | grep 8080
$ sudo ss -ntap | grep 1935

# Remove firewall 1935 & 8080
$ sudo ufw status numbered
$ sudo ufw delete [NUMBER]
~~~

## Install Restreamer
~~~
# Install restreamer for Laptop that support intel GPU
$ sudo docker run --detach --name restreamer --privileged \
--volume /opt/restreamer/config:/core/config \
--volume /opt/restreamer/data:/core/data \
--publish 8080:8080 --publish 8181:8181 --publish 1935:1935 --publish 1936:1936 --publish 6000:6000/udp \
datarhei/restreamer:vaapi-latest

# Open Firewall
$ sudo ufw allow 8080/tcp comment 'Allow restreamer http'
$ sudo ufw allow 8181/tcp comment 'Allow restreamer https'
$ sudo ufw allow 1935/tcp comment 'Allow restreamer rtmp'
$ sudo ufw allow 1936/tcp comment 'Allow restreamer rtmps'
$ sudo ufw allow 6000/udp comment 'Allow restreamer srt udp'
~~~

## Configure Restreamer for live streaming
~~~

~~~
