# Nifi-Docker
NiFi docker container customization from the [official Apache Nifi Docker Image](https://hub.docker.com/r/apache/nifi)

### 1. Build Docker
Pull the latest version of Apache Nifi Docker, build and run

```shell
$ docker pull apache/nifi:1.16.1
$ docker run --name nifi -e NIFI_WEB_HTTP_PORT='8080' -p 8080:8080 -d apache/nifi:1.16.1 --restart=always 
```
Entrar em http://localhost:8080/ pela aba anônima.

### 2. Download Connectors and Add Timezone

Dive into container shell, add timezone and download JARs & NARs
```shell
$ docker exec --user="root" -it nifi /bin/bash
nifi@container_id:/opt/nifi/nifi-current$ echo "java.arg.8=-Duser.timezone=America/Sao_Paulo" >> conf/bootstrap.conf
nifi@container_id:/opt/nifi/nifi-current$ cd lib
nifi@container_id:/opt/nifi/nifi-current/lib$ wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.29/mysql-connector-java-8.0.29.jar
nifi@container_id:/opt/nifi/nifi-current/lib$ wget https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc10/19.14.0.0/ojdbc10-19.14.0.0.jar
nifi@container_id:/opt/nifi/nifi-current/lib$ wget https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.5/postgresql-42.3.5.jar
nifi@container_id:/opt/nifi/nifi-current/lib$ wget https://repo1.maven.org/maven2/org/apache/nifi/nifi-kite-nar/1.15.3/nifi-kite-nar-1.15.3.nar
nifi@container_id:/opt/nifi/nifi-current/lib$ wget https://s3.amazonaws.com/rds-downloads/rds-ca-2019-root.pem
```
- Updated JDBC PostgreSQL Driver at https://jdbc.postgresql.org/download.html
- Updated JDBC Oracle Driver at https://www.oracle.com/database/technologies/appdev/jdbc-downloads.html
- Updated JDBC MySQL Driver at https://dev.mysql.com/downloads/connector/j/

### 3. Restart Nifi Web Service

Inside container, stop the service. It will kick you out of the container.

```shell
nifi@container_id:/opt/nifi/nifi-current$ ./bin/nifi.sh stop
```

Restart you container
```shell
$ docker start nifi
```

Update the restart policy
```shell
$ docker update --restart=always nifi
```

Read current restart policy
```shell
docker inspect -f "{{ .HostConfig.RestartPolicy }}" nifi
```

### 4. Access Nifi Web Service

Now you can access your Nifi web service at http://localhost:8080/nifi/

If you are in a VPN environment remember to tunnel 8080 port
```shell
$ ssh -f user@ip_address -L 8080:localhost:8080 -N
```
[How to tunnel in Putty](https://blog.devolutions.net/2017/4/how-to-configure-an-ssh-tunnel-on-putty) (for Windows users)

### 5. Basic Nifi Usage

This wiki page will cover the main proccess of NoHarm integration:
[Basic Nifi Usage](https://github.com/noharm-ai/nifi-docker/wiki/Basic-Nifi-Usage)

### 6. Testing JDBC Connection

- Download tool from http://jdbcsql.sourceforge.net/
- Run:  java -jar jdbcsql-1.0.zip -h hostname -p 1521 -U user -P pass -d service -m oracle "SELECT table_name FROM all_tables"

