# Nifi-Docker
NiFi docker container customization from the [official Apache Nifi Docker Image](https://hub.docker.com/r/apache/nifi)

### 1. Build Docker
Pull the latest version of Apache Nifi Docker, build and run

```shell
$ docker pull apache/nifi:1.18.0
```
Option 1: Run without authentication
```shell
$ docker run --name nifi -e NIFI_WEB_HTTP_PORT='8080' -p 8080:8080 -d apache/nifi:1.18.0 --restart=always
```
Open the URL http://localhost:8080/nifi.

Option 2: Run with authentication
```shell
docker run --name nifi -p 8443:8443 -d -e SINGLE_USER_CREDENTIALS_USERNAME=nifi_noharm -e SINGLE_USER_CREDENTIALS_PASSWORD=n0h@rmBR@2022   apache/nifi:1.18.0 --restart=always
```
Watch until the container is ready. You are looking for the "NiFi has started. The UI is available" message.
```shell  
docker logs -f nifi
```
Open the URL https://localhost:8443/nifi

### 2. Download Connectors and Add Timezone

Dive into container shell, add timezone and download JARs & NARs
```shell
$ docker exec --user="root" -it nifi /bin/bash
nifi@container_id:/opt/nifi/nifi-current$ echo "java.arg.8=-Duser.timezone=America/Sao_Paulo" >> conf/bootstrap.conf

nifi@container_id:/opt/nifi/nifi-current$ cd lib
```
Copy & Paste it
```shell
wget https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.29/mysql-connector-java-8.0.29.jar
wget https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc8/21.5.0.0/ojdbc8-21.5.0.0.jar
wget https://repo1.maven.org/maven2/org/postgresql/postgresql/42.4.0/postgresql-42.4.0.jar
wget https://repo1.maven.org/maven2/org/apache/nifi/nifi-kite-nar/1.15.3/nifi-kite-nar-1.15.3.nar
wget https://s3.amazonaws.com/rds-downloads/rds-ca-2019-root.pem
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

### Troubleshooting

#### a. Testing JDBC Connection

- Download tool from http://jdbcsql.sourceforge.net/
- Run:  java -jar jdbcsql-1.0.zip -h hostname -p 1521 -U user -P pass -d service -m oracle "SELECT table_name FROM all_tables"

#### b. Install SQL Plus

- Install:
```shell
sudo yum install -y yum install https://download.oracle.com/otn_software/linux/instantclient/216000/oracle-instantclient-basic-21.6.0.0.0-1.el8.x86_64.rpm
sudo yum install -y yum install https://download.oracle.com/otn_software/linux/instantclient/216000/oracle-instantclient-sqlplus-21.6.0.0.0-1.el8.x86_64.rpm
```
- Run:
```shell
sqlplus user/pass@localhost:1521/service
```

#### c. Repair "no space left" issue

Prune useless container and images:
```shell
$ docker system prune -a
```

Check fuller folders (if possible):
```shell
$ docker exec -t nifi sh -c "du -sh *"
```
Empty target folder (usually content_repository):
```shell
$docker exec -t nifi sh -c "rm content_repository/* -rf"
```
Restart nifi to repair empty folder:
```shell
docker exec -t nifi sh -c "./bin/nifi.sh restart"
```

You can do the same inside the container:
```shell
$ docker exec --user="root" -it nifi /bin/bash
```
#### d. Fix "docker.sock permission denied" issue
```shell
sudo chown $USER /var/run/docker.sock
```

#### e. Restarting Docker service if Docker is down:
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```shell
sudo service docker start
```

#### e. Changing Properties from HTTPS to HTTP:

 - https://github.com/apache/nifi/blob/main/nifi-docker/dockerhub/sh/start.sh#L53-L69
