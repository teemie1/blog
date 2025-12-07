# Install Sling Rebalancing Tool for CLN

## Download Sling from github
~~~
cd /data/lightningdm-plugins-available
wget https://github.com/daywalker90/sling/releases/download/v4.1.2/sling-v4.1.2-x86_64-linux-gnu.tar.gz
tar -xvf ./sling-v4.1.2-x86_64-linux-gnu.tar.gz
# For dynamic start plugin
lightning-cli plugin start /data/lightningdm-plugins-available/sling
~~~

## Change in config file
~~~
vi /data/lightningdm/config

plugin=/data/lightningdm-plugins-available/sling

systemctl restart lightningdm
~~~
