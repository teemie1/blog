# Install LNbits 1.2.1 by docker

~~~
docker pull lnbits/lnbits
wget https://raw.githubusercontent.com/lnbits/lnbits/main/.env.example -O .env
mkdir data
docker run --detach --publish 5000:5000 --name lnbits --volume ${PWD}/.env:/app/.env --volume ${PWD}/data/:/app/data lnbits/lnbits

~~~
