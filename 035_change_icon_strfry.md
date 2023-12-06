# Add Icon for strfry relay

## Edit nginx for favicon.ico
~~~
sudo nano /etc/nginx/sites-available/default
# Add Lines
    location = /favicon.ico {
    alias /var/www/html/bangkok.png;
    }

~~~
