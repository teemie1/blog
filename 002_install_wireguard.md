# วิธีติดตั้งและใช้งาน Wireguard บน Ubuntu Linux

## ติดตั้ง Wireguard บนทั้งสองเครื่อง
~~~
$ sudo apt install wireguard
~~~

## สร้าง key สำหรับ wireguard
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


