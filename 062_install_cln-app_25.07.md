# Install CLN Application 25.07

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
