# Install CLN Application 25.07

## Install latest NodeJS
~~~
sudo -iu lightningd

# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 20

# Verify the Node.js version:
node -v # Should print "v20.19.5".

# Verify npm version:
npm -v # Should print "10.8.2".

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
