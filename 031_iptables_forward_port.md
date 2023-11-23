# Configure port forwarding for new vpn server

~~~
$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 22022 -j DNAT --to-destination 10.7.0.12
$ sudo iptables -t nat -A POSTROUTING -o wg0 -p tcp --dport 22022 -d 10.7.0.12 -j SNAT --to-source 10.7.0.1


$ sudo netfilter-persistent save
$ sudo systemctl enable netfilter-persistent
~~~
