# Configure port forwarding for new vpn server

~~~
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8443 -j DNAT --to-destination 10.7.0.11
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 8443 -d 10.7.0.11 -j SNAT --to-source 10.7.0.1
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 22022 -j DNAT --to-destination 10.7.0.12
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 22022 -d 10.7.0.12 -j SNAT --to-source 10.7.0.1
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 8080 -j DNAT --to-destination 10.7.0.12
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 8080 -d 10.7.0.12 -j SNAT --to-source 10.7.0.1
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 4433 -j DNAT --to-destination 10.7.0.12
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 4433 -d 10.7.0.12 -j SNAT --to-source 10.7.0.1

$ sudo netfilter-persistent save
$ sudo systemctl enable netfilter-persistent
~~~
