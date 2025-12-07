# Install Summars summary tool for Core Lightning

## Download Summars and dynamic start plugin
~~~
cd /data/lightningdm-plugins-available
wget https://github.com/daywalker90/summars/releases/download/v6.0.0/summars-v6.0.0-x86_64-linux-gnu.tar.gz
tar -xvf summars-v6.0.0-x86_64-linux-gnu.tar.gz
lightning-cli plugin start /data/lightningdm-plugins-available/summars
~~~

## Change in config file
~~~
vi /data/lightningdm/config

plugin=/data/lightningdm-plugins-available/summars

systemctl restart lightningdm
~~~

## Sling command
~~~
clncli1 summars

~~~
