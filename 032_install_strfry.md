# Install strfry nostr relay

## Prerequsite for Ubuntu OS
~~~
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow OpenSSH
$ sudo ufw allow 80 comment 'Standard Webserver'
$ sudo ufw allow 443 comment 'SSL Webserver'
$ sudo ufw enable
$ sudo apt install fail2ban
$ sudo timedatectl set-timezone Asia/Bangkok
$ sudo nano .ssh/authorized_keys
# Fill public of my users

$ sudo nano /etc/ssh/sshd_config
# Uncomment and change authentication
   PasswordAuthentication no

# Remove sshd config file
$ sudo mv /etc/ssh/sshd_config.d/50-cloud-init.conf /tmp
$ sudo systemctl restart sshd
~~~


## Install strfry by git clone and compile
~~~
$ sudo -i
$ apt install -y git build-essential libyaml-perl libtemplate-perl libregexp-grammars-perl libssl-dev zlib1g-dev liblmdb-dev libflatbuffers-dev libsecp256k1-dev libzstd-dev
$ git clone https://github.com/hoytech/strfry && cd strfry/
$ git submodule update --init
$ make setup-golpe
$ make -j4
~~~

## Test running strfry
~~~
$ ./strfry relay
~~~

