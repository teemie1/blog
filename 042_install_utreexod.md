# Install Utreexod on Ubuntu

## Preparation
~~~
$ sudo -i
$ ufw allow 8333 comment 'Allow utreexod'

~~~

## Install Go
~~~
$ snap install go
$ go version
go version go1.22.2 linux/amd64

 
~~~

## Install Rust
~~~
$ curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
$ source $HOME/.cargo/env
$ rustc --version
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install build-essential
~~~

## Download utreexod & compile
~~~
$ sudo adduser --disabled-password --gecos "" utreexod
$ sudo usermod -a -G tee utreexod
$ sudo adduser tee utreexod
$ sudo -iu utreexod
$ git clone https://github.com/utreexo/utreexod
$ cd utreextod
$ make all
~~~

## Run utreexod
~~~
$ ./utreexod
Ctrl+C
~~~

## Systemd Configuration
~~~
$ sudo nano /etc/systemd/system/utreexod.service
[Unit]
Description=utreexod
Wants=network-online.target
After=network-online.target
 

[Service]
User=utreexod
Group=utreexod
Type=simple
ExecStart=/usr/local/bin/utreexod 

[Install]
WantedBy=multi-user.target

sudo systemctl enable utreexod
sudo systemctl start utreexod
sudo journalctl -u utreexod
~~~
