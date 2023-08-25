# Install Core Lightning 23.08 and Failback to NODE1

## Download Core Lightning 23.08
~~~
# Download Core Lightning Software and Checksum
$ cd /tmp
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/clightning-v23.08-Ubuntu-22.04.tar.xz
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/SHA256SUMS
$ wget https://github.com/ElementsProject/lightning/releases/download/v23.08/SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS
clightning-v23.08-Ubuntu-22.04.tar.xz: OK

# Download and verify key with gpg
$ wget https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/rustyrussell.txt
$ wget https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/cdecker.txt
$ wget https://raw.githubusercontent.com/ElementsProject/lightning/master/contrib/keys/niftynei.txt
$ gpg --import rustyrussell.txt
$ gpg --import cdecker.txt
$ gpg --import niftynei.txt
$ gpg --verify SHA256SUMS.asc

# Install Core Lightning
$ cd /
$ sudo tar -xvf /tmp/clightning-v23.08-Ubuntu-22.04.tar.xz    # this will extract lightningd binary to the system
$ lightningd --version
~~~


## Prepare environment for Core Lightning
~~~
$ sudo adduser --disabled-password --gecos "" lightningd
$ sudo usermod -a -G bitcoin,debian-tor lightningd
$ sudo adduser tee lightningd
$ sudo mkdir /data/lightningd
$ sudo mkdir /data/lightningd-plugins-available
$ sudo chown -R lightningd:lightningd /data/lightningd
$ sudo chown -R lightningd:lightningd /data/lightningd-plugins-available
$ sudo su - lightningd
$ ln -s /data/lightningd /home/lightningd/.lightning
$ ln -s /data/bitcoin /home/lightningd/.bitcoin
~~~


## 
~~~

~~~


## 
~~~

~~~


## 
~~~

~~~

