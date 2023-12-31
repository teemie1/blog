# Configure Node00 for Post-BTC Workshop Class

## Shutdown node00 template node.
~~~
$ sudo shutdown -h now
~~~

## Use linked clone script for clone new node vm
https://github.com/oliverbock/esxi-linked-clone/blob/master/README.md

 - Download script [clone.sh](https://raw.githubusercontent.com/oliverbock/esxi-linked-clone/master/clone.sh) & [deleteclone.sh](https://raw.githubusercontent.com/oliverbock/esxi-linked-clone/master/deleteclone.sh)
 - Upload to esxi
 - Copy script to the folder beside template VM
 - Take snapshot on template VM.

## Run link clone in esxi
~~~
$ ./clone.sh Sakchai_debian11/ node01
~~~

## Open console of nodeXX
~~~
$ sudo /scripts/config_nodeXX.sh
$ sudo shutdown -r now
~~~

## Install system disk for raspiblitz
Connect via VPN
~~~
# download the build script
$ wget https://raw.githubusercontent.com/rootzoll/raspiblitz/dev/build_sdcard.sh
# run
$ sudo bash build_sdcard.sh -f false -b dev -d headless -t false -w off

$ sudo shutdown -r now
~~~

## Configure raspiblitz
 - Login with admin user
 - Select fresh install and delete all existing data
 - Fill Password A,B,C,D --> all the same
 - Wait until finish system Setup
 - Open another terminal windows
~~~
$ sudo systemctl stop bitcoind
$ sudo rm -rf /mnt/hdd/bitcoin/blocks
$ sudo rm -rf /mnt/hdd/bitcoin/chainstate
$ sudo ln -s /data/bitcoin/blocks /mnt/hdd/bitcoin/blocks
$ sudo ln -s /data/bitcoin/chainstate /mnt/hdd/bitcoin/chainstate
$ sudo ln -s /data/bitcoin/indexes /mnt/hdd/bitcoin/indexes
$ sudo chown -R bitcoin:bitcoin /mnt/hdd/bitcoin
$ sudo chown -R bitcoin:bitcoin /data/bitcoin
$ sudo chmod -R 777 /mnt/hdd/bitcoin
$ sudo chmod -R 777 /data/bitcoin
$ sudo systemctl start bitcoind
$ sudo tail -f /mnt/hdd/bitcoin/debug.log
~~~
