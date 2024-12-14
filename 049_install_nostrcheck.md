# Install Nostrcheck Server for Media Server

## Download github from nostrcheck and create docker container
~~~
sudo -i
cd /data
git clone https://github.com/quentintaranpino/nostrcheck-server
ln -s /data/nostrcheck-server /root/nostrcheck-server
cd nostrcheck-server
docker-compose up -d
~~~
