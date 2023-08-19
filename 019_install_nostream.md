# Install nostream for Paid Public Nostr Relay

## Prerequisite Standalone setup
 - PostgreSQL 14.0
 - Redis
 - Node v18
 - Typescript


## Install Prerequisite
~~~
# Update deps & install nodejs, npm, nginx, certbot
$ sudo apt update
$ sudo apt install nodejs npm nginx certbot python3-certbot-nginx

# Setup new `nostream` user (don't run nostream on root)
$ sudo adduser --gecos "" --disabled-password nostr
$ sudo adduser tee nostr

~~~

## Install nostream
~~~
# Change to nostream user
$ sudo -i -u nostr

# Clone `nostream` repo
$ git clone https://github.com/Cameri/nostream.git

# Open a TMUX session
# (to be able to detach and maintain process running)
$ tmux

# Start the relay
$ ./scripts/start

# You want to start the relay once such that all Docker images are downloaded/built, and the default settings.yaml file is automatically copied over.

# Stop the relay (you will see the NOSTREAM logo once it's running)
Ctrl + C (you can use ./scripts/stop as well)

# Edit the settings file to your liking
# (see Settings.yaml Configuration below)

# Add local.env file to root
touch local.env

# Edit local.env file and add ZEBEDEE_API_KEY and SECRET
# SECRET is a 128bit random hash
nano local.env

ZEBEDEE_API_KEY="your API key goes here"
SECRET="your SECRET goes here"

# You may need to add a `env_file` property to docker-compose.yml
env_file
  - local.env

# Restart Nostream
./scripts/start

# To detach from the TMUX session
Ctrl+B  +  D
# To re-attach to the TMUX session
tmux a
~~~

## Running as a Service
~~~
$ nano /etc/systemd/system/nostream.service

# Note: replace "User=..." with your username, and
# "/home/nostr/nostream" with the directory where you cloned the repo.

[Unit]
Description=Nostr TS Relay
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=5
User=nostr
WorkingDirectory=/home/nostr/nostream
ExecStart=/home/nostr/nostream/scripts/start
ExecStop=/home/nostr/nostream/scripts/stop

[Install]
WantedBy=multi-user.target
~~~
~~~
$ systemctl enable nostream
$ systemctl start nostream
$ journalctl -fu nostream
~~~

## Initializing the database
~~~
$ psql -h $DB_HOST -p $DB_PORT -U $DB_USER -W
postgres=# create database nostr_ts_relay;
postgres=# quit
~~~
~~~
$ redis-cli
127.0.0.1:6379> CONFIG SET requirepass "nostr_ts_relay"
OK
127.0.0.1:6379> AUTH nostr_ts_relay
Ok
~~~
~~~
$ git clone git@github.com:Cameri/nostream.git
$ cd nostream
$ npm install -g knex
$ npm install
$ NODE_OPTIONS="-r dotenv/config" npm run db:migrate
$ mkdir .nostr
$ cp resources/default-settings.yaml .nostr/settings.yaml

~~~
