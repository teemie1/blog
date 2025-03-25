# Install pgAdmin Tool for PostgreSQL

 - Install the public key for the PgAdmin4 repository:
~~~
curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add
~~~

 - Create the repository configuration file:
~~~
sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/focal/ pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'
~~~

 - Choose your preferred mode for PgAdmin4 installation:
For both desktop and web modes:
~~~
sudo apt install pgadmin4
~~~

# Alternative install pgAdmin with docker container

~~~
docker pull dpage/pgadmin4
docker run --restart always --name pgadmin-container -p 5050:80 -e PGADMIN_DEFAULT_EMAIL=teemie@satsdays.com -e PGADMIN_DEFAULT_PASSWORD=XXXXXX -d dpage/pgadmin4
~~~
