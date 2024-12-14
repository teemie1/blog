# Install Nostrcheck Server for Media Server

## Download compost file from nostrcheck
~~~
sudo -i
cd /data
git clone https://github.com/quentintaranpino/nostrcheck-server
ln -s /data/nostrcheck-server /root/nostrcheck-server
cd nostrcheck-server
docker-compose up -d
~~~
