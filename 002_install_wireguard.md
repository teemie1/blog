# วิธีติดตั้งและใช้งาน Wireguard บน Ubuntu Linux

## ติดตั้ง Wireguard บนทั้งสองเครื่อง
~~~
$ sudo apt install wireguard
~~~

## สร้าง key สำหรับ wireguard ทั้งสองเครื่อง
~~~
$ sudo -i
$ mkdir -m 0700 /etc/wireguard/
$ cd /etc/wireguard/
$ umask 077; wg genkey | tee privatekey | wg pubkey > publickey
$ ls -l privatekey publickey
$ cat privatekey
## Please note down the private key ##
$ cat publickey
## Please note down the public key ##
~~~

## แก้ไขไฟล์ wg0.conf บน server
~~~
$ sudo nano /etc/wireguard/wg0.conf
[Interface]
Address = 10.8.0.1/24
SaveConfig = true
PostUp = ufw route allow in on %i out on eth0
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

PreDown = ufw route delete allow in on %i out on eth0
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

ListenPort = 51820
PrivateKey = [PRIVATE KEY OF SERVER]

[Peer]
PublicKey = [PUBLIC KEY OF CLIENT]
AllowedIPs = 10.8.0.21/32

~~~
## แก้ไขไฟล์ /etc/sysctl.conf บน server
~~~
$ vi /etc/sysctl.conf
# uncomment
net.ipv4.ip_forward=1
~~~

## แก้ไขไฟล์ wg0.conf บน client
~~~
$ sudo nano /etc/wireguard/wg0.conf
[Interface]
PrivateKey = [PRIVAT KEY OF CLIENT]
Address = 10.8.0.21/24

[Peer]
PublicKey = [PUBLIC KEY OF SERVER]
AllowedIPs = 10.8.0.0/24
Endpoint = [IP OF SERVER]:51820
PersistentKeepalive = 25

~~~
## Start wireguard ทั้งสองเครื่อง

~~~
$ sudo systemctl enable wg-quick@wg0
$ sudo systemctl start wg-quick@wg0
$ sudo systemctl status wg-quick@wg0
~~~

## เช็คสถานะการเชื่อมต่อ wireguard
~~~
$ wg
interface: wg0
  public key: [public key]
  private key: (hidden)
  listening port: 51820

peer: [public key of peer]
  endpoint: 114.10.9.198:32114
  allowed ips: 10.8.1.0/24
  latest handshake: 1 minute, 7 seconds ago
  transfer: 93.66 KiB received, 28.94 KiB sent

$ ping 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=28.6 ms
64 bytes from 10.8.0.1: icmp_seq=2 ttl=64 time=28.2 ms
64 bytes from 10.8.0.1: icmp_seq=3 ttl=64 time=29.3 ms

~~~
ทั้งสองเครื่องสามารถเชื่อมต่อ wireguard กันสำเร็จเรียบร้อย ใช้ IP:10.8.0.x ในการติดต่อหากันได้สำเร็จ

## Add peer online without restart wireguard
~~~
wg set wg0 peer "[PUBLIC KEY OF CLIENT]" allowed-ips 10.8.0.2/32
ip -4 route add 10.8.0.2/32 dev wg0
~~~
