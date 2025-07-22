# Upgrade LNbits 1.2.1

## Stop LNbits
~~~
sudo systemctl stop lnbits
sudo su - lnbits
~~~
## Upgrade LNbits v1.2.1
~~~
cd /home/lnbits/lnbits
git fetch
git reset --hard HEAD
git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
> v1.2.1
git checkout v1.2.1
poetry install --only main
exit
~~~
## Restart LNbits
~~~
sudo systemctl start lnbits
~~~
