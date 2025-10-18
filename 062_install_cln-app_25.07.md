# Install CLN Application 25.07

## Install latest NodeJS
~~~
curl -sL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm update -g npm
~~~

## Download CLN App
~~~
sudo -iu lightningd
wget https://github.com/ElementsProject/cln-application/archive/refs/tags/v25.07.tar.gz
tar -xzf v25.07.tar.gz
~~~

## Install and Compile
~~~
cd cln-application-25.07
npm ci
npm run build
npm prune --omit=dev
~~~
