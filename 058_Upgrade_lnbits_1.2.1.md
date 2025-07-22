# Upgrade LNbits 1.2.1

## Stop LNbits
~~~
sudo systemctl stop lnbits
sudo su - lnbits
~~~
## Install python3.13
~~~
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.13
python3.13 --version
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
