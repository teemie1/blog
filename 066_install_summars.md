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

## Summars command
~~~
clncli1 summars
lightning-cli summars summars-forwards=200 summars-pays=300 summars-invoices=300
lightning-cli summars summars-columns=GRAPH_SATS,SCID summars-style=empty summars-sort-by=IN_SATS
lightning-cli summars summars-columns=GRAPH_SATS,SCID,ALIAS,BASE,PPM summars-style=empty summars-sort-by=IN_SATS
~~~
