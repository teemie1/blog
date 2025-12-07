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

## Sling command
~~~
clncli1 sling-version

# Push Channel
clncli2 sling-job -k scid=2638506x14x0 direction=push amount=100000 maxppm=300 outppm=0 target=0.4

# Pull Channel
clncli1 sling-job -k scid=2667713x5x2 direction=pull amount=100000 maxppm=300 outppm=1000 target=0.4

clncli2 sling-jobsettings
clncli2 sling-go
clncli2 sling-stats
clncli2 sline-stop
~~~
