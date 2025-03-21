# Install Ohttp Relay 0.0.10 by docker container

Ref: https://hub.docker.com/layers/dangould/ohttp-relay/0.0.10-rc0/images/sha256-2c9ccf85d6101195c9b97eafcf6b561c842d085754a5172a4d5af9f4b60ca27c

## Install ohttp-relay image and run
~~~
docker pull dangould/ohttp-relay:0.0.10-rc0
docker run -d -e PORT=4000 -e GATEWAY_ORIGIN="https://payjo.in:443" --restart unless-stopped --name bob-ohttp-relay -p 4000:4000 dangould/ohttp-relay:0.0.10-rc0
~~~
