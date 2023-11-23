# Install strfry nostr relay

## Prerequsite for Ubuntu OS
~~~

~~~


## Install strfry by git clone and compile
~~~
$ sudo -i
$ apt install -y git build-essential libyaml-perl libtemplate-perl libregexp-grammars-perl libssl-dev zlib1g-dev liblmdb-dev libflatbuffers-dev libsecp256k1-dev libzstd-dev
git clone https://github.com/hoytech/strfry && cd strfry/
git submodule update --init
make setup-golpe
make -j4
~~~

## Test running strfry
~~~
$ ./strfry relay
~~~

