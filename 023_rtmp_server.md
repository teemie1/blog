# Create RTMP Streaming Server on Ubuntu 22.04

This streaming server already installed NGINX.
## Installing and Configuring Nginx-RTMP
~~~
$ sudo apt update
$ sudo apt install libnginx-mod-rtmp
$ sudo nano /etc/nginx/nginx.conf
~~~

~~~
. . .
rtmp {
        server {
                listen 1935;
                chunk_size 4096;
#                allow publish 127.0.0.1;
                allow publish all;

                application live {
                        live on;
                        record off;
                }
        }
}
~~~
~~~
$ sudo ufw allow 1935/tcp
$ sudo systemctl reload nginx.service
~~~

## Streaming Video to Your Server via OBS (Optional)
Download (OBS)[https://cdn-fastly.obsproject.com/downloads/OBS-Studio-29.1.3-Full-Installer-x64.exe] and start stream from the following config.
~~~
Streaming Service: Custom
Server: rtmp://your_domain/live
Play Path/Stream Key: obs_stream
~~~
To see the streaming use VLC and "Open Network Stream..." to "rtmp://teemie1-relay.duckdns.org/live/obs_stream

## Adding Monitoring to Your Configuration (Optional)
~~~
$ sudo nano /etc/nginx/sites-available/rtmp
server {
    listen 8080;
    server_name  localhost;

    # rtmp stat
    location /stat {
        rtmp_stat all;
        rtmp_stat_stylesheet stat.xsl;
    }
    location /stat.xsl {
        root /var/www/html/rtmp;
    }

    # rtmp control
    location /control {
        rtmp_control all;
    }
}

$ sudo mkdir /var/www/html/rtmp
$ sudo cp /usr/share/doc/libnginx-mod-rtmp/examples/stat.xsl /var/www/html/rtmp/stat.xsl
$ sudo ufw allow from 10.8.0.0/24 to any port http-alt comment 'Allow VPN Client to connect'
$ sudo ufw allow from 192.168.1.0/24 to any port http-alt comment 'Allow Local Client to connect'
$ sudo ln -s /etc/nginx/sites-available/rtmp /etc/nginx/sites-enabled/rtmp
$ sudo systemctl reload nginx.service

~~~
You should now be able to go to http://teemie1-relay.duckdns.org:8080/stat

## Creating Modern Streams for Browsers (Optional)
~~~
$ sudo nano /etc/nginx/nginx.conf
. . .
rtmp {
        server {
. . .
                application live {
                    	live on;
                    	record off;
                        hls on;
                        hls_path /var/www/html/stream/hls;
                        hls_fragment 3;
                        hls_playlist_length 60;

                        dash on;
                        dash_path /var/www/html/stream/dash;
                }
        }
}
. . .

$ sudo nano /etc/nginx/sites-available/rtmp
. . .
server {
    listen 8088;

    location / {
        add_header Access-Control-Allow-Origin *;
        root /var/www/html/stream;
    }
}

types {
    application/dash+xml mpd;
}

$ sudo ufw allow 8088/tcp
$ sudo mkdir /var/www/html/stream
$ sudo systemctl reload nginx

~~~
You should now have an HLS stream available at http://teemie1-relay.duckdns.org:8088/hls/obs_stream.m3u8 and a DASH stream available at http://teemie1-relay.duckdns.org:8088/dash/stream.mpd. These endpoints will generate any necessary metadata on top of your RTMP video feed in order to support modern APIs.

## Add authentication
~~~
$ sudo vi /etc/nginx/nginx.conf
...
rtmp {
        server {
...
                application live {
                        live on;
                        on_publish http://localhost:8080/auth;
   ...
                }
        }
}
...
$ sudo nano /etc/nginx/sites-available/rtmp
...
    listen 8080;
    server_name  localhost;

    # rtmp authentication
    location /auth {
        if ($arg_pwd = 'rightshift') {
            return 200;
            }
            return 401;
    }


$  systemctl restart nginx.service
~~~
## Streaming Video to Your Server via OBS with password

~~~
Streaming Service: Custom
Server: rtmp://teemie1-relay.duckdns.org/live
Play Path/Stream Key: obs_stream?pwd=rightshift
~~~
To see the streaming use VLC and "Open Network Stream..." to "rtmp://teemie1-relay.duckdns.org/live/obs_stream

