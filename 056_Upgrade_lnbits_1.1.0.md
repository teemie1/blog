# Upgrade LNbits 1.1.0

## Stop LNbits
~~~
sudo systemctl stop lnbits
sudo su - lnbits
~~~
##
~~~
cd /home/lnbits/lnbits
git fetch
git reset --hard HEAD
git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
> v1.1.0
git checkout v1.1.0
poetry install --only main
exit
~~~
