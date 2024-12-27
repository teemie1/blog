# Install NGINX for Payjoin Relay

~~~
$ sudo apt install nginx certbot python3-certbot-nginx

# Delete the default nginx settings file
$ sudo rm -rf /etc/nginx/sites-available/default

# Paste in new settings file contents (see heading NGINX Settings below)
$ sudo nano /etc/nginx/sites-available/default
server {
    server_name payjo.in;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}


# Restart nginx
$ sudo systemctl start nginx

# Map DNS A record to IP of VM machine (see DNS Settings below)

# Request SSL cert from letsencrypt/certbot
$ sudo certbot --nginx -d payjo.in

# Change port 80 also
$ sudo nano /etc/nginx/sites-available/default
server {
#    if ($host = payjo.in) {
#        return 301 https://$host$request_uri;
#    } # managed by Certbot


    server_name payjo.in;
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:8080;
    }
#    return 404; # managed by Certbot


}


$ sudo systemctl restart nginx
~~~

## Forward port
~~~
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 10.8.0.205
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 80 -d 10.8.0.205 -j SNAT --to-source 10.8.0.1
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 10.8.0.205
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 443 -d 10.8.0.205 -j SNAT --to-source 10.8.0.1

$ sudo netfilter-persistent save
$ sudo systemctl enable netfilter-persistent

~~~

## certbot command
~~~
$ certbot certificates
$ certbot renew
~~~
