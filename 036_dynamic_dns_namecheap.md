# Dynamic DNS with Namecheap

Ref: https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient/

## Install and Configure ddclient
~~~
$ sudo apt install ddclient
$ sudo vi /etc/ddclient.conf
use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com
login=satsdays.com
password=[PASSWORD FROM NAMECHEAP]
@,vpn,restreamer,nostr

$ systemctl restart ddclient.service
~~~
Password from namecheap: Look at Dashboard --> Manage (satsdays.com) --> Advanced DNS --> Dynamic DNS Password
