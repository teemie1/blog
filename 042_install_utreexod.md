# Install Utreexod on Ubuntu

## Install Go
~~~
$ snap install go
$ go version
go version go1.22.2 linux/amd64

 
~~~

## Install Rust
~~~
$ curl --proto '=https' --tlsv1.3 https://sh.rustup.rs -sSf | sh
$ rustc --version
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install build-essential
~~~

## Download utreexod & compile
~~~
$ git clone https://github.com/utreexo/utreexod
$ cd utreextod
$ make all
~~~

## Run utreexod
~~~
$ ./utreexod
~~~
