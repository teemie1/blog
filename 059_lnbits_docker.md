# Install LNbits 1.2.1 by docker
# Not Work
~~~
docker pull lnbits/lnbits
docker run --detach --publish 5000:5000 --name lnbits --volume /mnt/data/lnbits/.env:/app/.env --volume /mnt/data/lnbits/data/:/app/data lnbits/lnbits

~~~
