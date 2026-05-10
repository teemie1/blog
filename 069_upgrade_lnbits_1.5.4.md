# Upgrade LNbits 1.5.4

## Stop LNbits
~~~
sudo systemctl stop lnbits
sudo su - lnbits
~~~
## Install UV
~~~
sudo -iu lnbits
cd lnbits
curl -LsSf https://astral.sh/uv/install.sh | sh
~~~

## Upgrade LNbits v1.5.4
~~~
cd /home/lnbits/lnbits
git fetch
git reset --hard HEAD
git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
> v1.5.4
git checkout v1.5.4
uv sync --all-extras
exit
~~~
## Restart LNbits
~~~
vi /etc/systemd/system/lnbits.service

ExecStart=/home/lnbits/.local/bin/uv run lnbits --port 5000 --host 0.0.0.0 --debug --reload

systemctl daemon-reload
sudo systemctl start lnbits
~~~
