# Upgrade RTL 0.15.8

## Stop RTL
~~~
# sudo systemctl stop rtl
~~~

## Upgrade RTL
~~~
# sudo -iu rtl
# cd ~/RTL
# git fetch
# git reset --hard HEAD
# git tag | grep -E "v[0-9]+.[0-9]+.[0-9]+$" | sort --version-sort | tail -n 1
v0.15.8
# git checkout v0.15.8

# npm install --omit=dev --legacy-peer-deps
# npm install -g npm@11.14.1
# exit
~~~

## Start RTL
~~~
# systemctl start rtl
~~~
