# Use socat for port forwarding

~~~
$ sudo nano /etc/systemd/system/payjoin-relay-ssh.service

#add the following to the file

[Unit]
Description=payjoin-relay-ssh
After=network.target

[Service]
ExecStart=/usr/bin/socat tcp4-LISTEN:10022,reuseaddr,fork TCP:10.8.0.205:22

[Install]
WantedBy=multi-user.target
~~~

~~~
$ sudo nano /etc/systemd/system/payjoin-relay-https.service

#add the following to the file

[Unit]
Description=payjoin-relay-https
After=network.target

[Service]
ExecStart=/usr/bin/socat tcp4-LISTEN:443,reuseaddr,fork TCP:10.8.0.205:443

[Install]
WantedBy=multi-user.target
~~~
~~~
$ sudo nano /etc/systemd/system/payjoin-relay-http.service

#add the following to the file

[Unit]
Description=payjoin-relay-http
After=network.target

[Service]
ExecStart=/usr/bin/socat tcp4-LISTEN:80,reuseaddr,fork TCP:10.8.0.205:80

[Install]
WantedBy=multi-user.target
~~~

~~~
$ sudo systemctl enable payjoin-relay-ssh.service
$ sudo systemctl enable payjoin-relay-http.service
$ sudo systemctl enable payjoin-relay-https.service

$ sudo systemctl start payjoin-relay-ssh.service
$ sudo systemctl start payjoin-relay-http.service
$ sudo systemctl start payjoin-relay-https.service

# These are for reference
# socat TCP-LISTEN:10022,fork TCP:10.8.0.205:22
# socat TCP-LISTEN:80,fork TCP:10.8.0.205:80
# socat TCP-LISTEN:443,fork TCP:10.8.0.205:443
~~~
