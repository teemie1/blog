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
ListenPort = 51820
PrivateKey = [privatekey of server]

[Peer]
PublicKey = [publickey of client]
AllowedIPs = 10.8.0.2/24
~~~

## แก้ไขไฟล์ wg0.conf บน client
~~~
$ sudo nano /etc/wireguard/wg0.conf
[Interface]
Address = 10.8.0.2/24
PrivateKey = [privatekey of client]

[Peer]
PublicKey = [publickey of server]
Endpoint = [ip of server]:51820
AllowedIPs = 10.8.0.0/24
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
  public key: /gxiXxR5rB9Uxul28b+qnHEfa7FF2d2sHkVCYteiuCe=
  private key: (hidden)
  listening port: 51820

peer: bqCbtP5dBMNl1aLIs16o9kzGQE1u0yNgftIv0bUhn1Q=
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
