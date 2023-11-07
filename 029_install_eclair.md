# Install Eclair Lightning Server on Raspiblitz

## Install OpenJava
Reference: [link](https://adoptium.net/installation/linux/)
~~~
$ sudo -i
$ apt install -y wget apt-transport-https
$ mkdir -p /etc/apt/keyrings
$ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
$ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
$ apt update 
$ apt install temurin-11-jdk
~~~

## Download Eclair and Configuration Files
~~~

~~~

## Start Eclair
~~~

~~~

## Configure Systemd
~~~

~~~
