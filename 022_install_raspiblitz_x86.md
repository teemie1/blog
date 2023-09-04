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
~~~

~~~
