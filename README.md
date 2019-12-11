# Nifi-Docker
NiFi docker container customization from the [official Apache Nifi Docker Image](https://hub.docker.com/r/apache/nifi)

### 1. Build Docker
Pull the latest version of Apache Nifi Docker, buuild and run

```shell
$ docker pull apache/nifi
$ docker run --name nifi -p 8080:8080 -d apache/nifi:latest
```

### 2. Download Connectors and Add Timezone

Dive into container shell, add timezone and download JARs & NARs
```shell
$ docker exec -i -t nifi /bin/bash
nifi@container_id:/opt/nifi/nifi-current$ echo "java.arg.8=-Duser.timezone=America/New_York" >> conf/bootstrap.conf
nifi@container_id:/opt/nifi/nifi-current$ cd lib
nifi@container_id:/opt/nifi/nifi-current$ wget https://hostname/nifi/mysql-connector-java-8.0.18.jar
nifi@container_id:/opt/nifi/nifi-current$ wget https://hostname/nifi/ojdbc8.jar
nifi@container_id:/opt/nifi/nifi-current$ wget https://jdbc.postgresql.org/download/postgresql-42.2.9.jar
nifi@container_id:/opt/nifi/nifi-current$ wget https://repo1.maven.org/maven2/org/apache/nifi/nifi-kite-nar/1.10.0/nifi-kite-nar-1.10.0.nar
```
- Update JDBC PostgreSQL Driver at https://jdbc.postgresql.org/download.html
- Update JDBC Oracle Driver at https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html
- Update JDBC MySQL Driver at https://dev.mysql.com/downloads/connector/j/

### 3. Restart Nifi Web Service

Inside container, stop the service. It will kick you out of the container.

```shell
nifi@container_id:/opt/nifi/nifi-current$ ./bin/nifi.sh stop
```

Restart you container: Find the container ID and start it again.
```shell
$ docker ps -a
$ docker start nifi
```

### 4. Access Nifi Web Service

Now you can access your Nifi web service at http://localhost:8080/nifi/

If you are in a VPN environment remember to tunnel 8080 port
```shell
$ ssh user@ip_address -L8080:localhost:8080
```
[How to tunnel in Putty](https://blog.devolutions.net/2017/4/how-to-configure-an-ssh-tunnel-on-putty) (for Windows users)

### 5. Basic Nifi Usage

This wiki page will cover the main proccess of NoHarm integration:
[Basic Nifi Usage](https://github.com/noharm-ai/nifi-docker/wiki/Basic-Nifi-Usage)

### 6. Testing JDBC Connection

- Download http://jdbcsql.sourceforge.net/
- Run:  java -jar jdbcsql-1.0.zip -h hostname -p 1521 -U user -P pass -d service -m oracle "SELECT table_name FROM all_tables"

