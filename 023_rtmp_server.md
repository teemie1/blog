# Create RTMP Streaming Server on Ubuntu 22.04

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
                allow publish 127.0.0.1;
                deny publish all;

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

## Sending Video to Your RTMP Server
~~~
sudo apt install python3-pip
sudo pip install youtube-dl
youtube-dl https://www.youtube.com/watch?v=iom_nhYQIYk
sudo apt install ffmpeg
ffmpeg -re -i "Introducing App Platform by DigitalOcean-iom_nhYQIYk.mkv" -c:v copy -c:a aac -ar 44100 -ac 1 -f flv rtmp://localhost/live/stream
~~~

## Sending Video to Your RTMP Server
~~~
sudo nano /etc/nginx/nginx.conf
. . .
                allow publish 127.0.0.1;
                allow publish your_local_ip_address;
                deny publish all;
. . .

sudo systemctl reload nginx.service
~~~

## Sending Video to Your RTMP Server
~~~
sudo nano /etc/nginx/sites-available/rtmp
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

sudo mkdir /var/www/html/rtmp
sudo gunzip -c /usr/share/doc/libnginx-mod-rtmp/examples/stat.xsl.gz > /var/www/html/rtmp/stat.xsl
sudo ufw allow from your_ip_address to any port http-alt
sudo ln -s /etc/nginx/sites-available/rtmp /etc/nginx/sites-enabled/rtmp
sudo systemctl reload nginx.service

~~~

## Creating Modern Streams for Browsers (Optional)
~~~
sudo nano /etc/nginx/nginx.conf
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

sudo nano /etc/nginx/sites-available/rtmp
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

sudo ufw allow 8088/tcp
sudo mkdir /var/www/html/stream
sudo systemctl reload nginx

~~~
You should now have an HLS stream available at http://your_domain:8088/hls/stream.m3u8 and a DASH stream available at http://your_domain:8088/dash/stream.mpd. These endpoints will generate any necessary metadata on top of your RTMP video feed in order to support modern APIs.

## 
~~~

~~~

