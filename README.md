# Nifi-Docker
NiFi docker container customization from the official source

### 1. Build Docker
Pull the latest version of Apache Nifi Docker, buuild and run

```shell
$ docker pull apache/nifi
$ docker build -t apache/nifi:latest
$ docker run --name nifi -p 8080:8080 -d apache/nifi:latest
```

### 2. Download Connectors and Add Timezone

Dive into container shell, add timezone and download JARs & NARs
```shell
$ docker exec -i -t container_id /bin/bash
nifi@container_id:/opt/nifi/nifi-current$ echo "java.arg.8=-Duser.timezone=America/New_York" >> conf/bootstrap.conf
nifi@container_id:/opt/nifi/nifi-current$ cd lib
nifi@container_id:/opt/nifi/nifi-current$ wget http://hostname/mysql-connector-java-8.0.18.jar
nifi@container_id:/opt/nifi/nifi-current$ wget http://hostname/ojdbc8.jar
nifi@container_id:/opt/nifi/nifi-current$ wget https://repo1.maven.org/maven2/org/apache/nifi/nifi-kite-nar/1.10.0/nifi-kite-nar-1.10.0.nar
```

### 3. Restart Nifi Web Service

Inside container, stop the service. It will kick you out of the container.

```shell
nifi@container_id:/opt/nifi/nifi-current$ ./bin/nifi.sh stop
```

Restart you container: Find the container ID and start it again.
```shell
$ docker ps -a
$ docker start container_id
```

### 4. Access Nifi Web Service

Now you can access your Nifi web service at http://localhost:8080/nifi/

If you are in a VPN environment remember to tunnel 8080 port
```shell
$ ssh user@ip_address -L8080:localhost:8080
```
[How to tunnel in Putty](https://blog.devolutions.net/2017/4/how-to-configure-an-ssh-tunnel-on-putty) (for Windows users)
