# Connect Zeus Mobile App to Core Lightning

## Get magaroon file in hex
~~~
$ sudo su - rtl
$ xxd -ps -u -c 1000  access.macaroon
# Copy hex as text and use in Zeus
~~~
## Open CL-Rest API Port in FW
~~~
$ sudo ufw allow 3092/tcp comment 'allow CLREST for zeus'
~~~

## Connect zeus to core lightning
~~~
Node Interface: Core Lightning (c-lightning-REST)
Host:           10.8.0.2
Macaroon:       [From xxd command]
REST Port:      3092
Use Tor:        No
Certificate Verification: No
~~~
