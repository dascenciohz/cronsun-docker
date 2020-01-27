# CRONSUN

**Cronsun** is a distributed cron-style job system. It's similar to crontab.

With Cronsun you can schedule your work to run in the background and at the intervals of time you want. You can schedule jobs in PHP, Python, Node, Shell, MySQL and any other you want, you just have to configure a work node.

Cronsun uses [ETCD](https://github.com/etcd-io/etcd) in Cluster mode to store all the data corresponding to the scheduled jobs. Each output of the scheduled jobs is stored in [Mongo DB](https://www.mongodb.com/es).

# SECURITY

## Task Execution - Users

Scheduled tasks are executed by specific users. If these users are not allowed by Cronsun, they will not be executed.
This is why, if you want to execute tasks as a "custom" user, you must allow it in the security.json file.

```
{
    "open": false,
    "users": [
        "www-data", "node", "myuser", "mycustomuser"
    ],
    "ext": [
        ".sh", ".py"
    ]
}
```

**NOTE** Always try to avoid the root user to execute your work.

## DigestAuth - Web Security
The [DigestAuth - Traefik](https://docs.traefik.io/middlewares/digestauth) middleware is a quick way to restrict access to your services to known users.

**htdigest** command is used to create and update the flat-files used to store usernames, realm and password for digest authentication of HTTP users.

1. Create passwd file with realm and username.

```
# Example to execute
htdigest -c passwdfile realm username

# Execute
htdigest -c passwdfile cronsun cronsun
```

2. Read passwdfile to view encrypted authentication

```
cat passwdfile

# output example
cronsun:cronsun:f9cd937852fe269683e8dac85d4744e9
```

**NOTE** By default the username and password is: cronsun

## HTTPS - Let's Encrypt

You can configure Traefik to use an ACME provider (like Let's Encrypt) for automatic certificate generation.

To configure cronsun under HTTPS you just have to uncomment the following lines in your docker-compose.yaml file:

```
#- --certificatesResolvers.sample.acme.email=${INGRESS_ADMIN_MAIL}
#- --certificatesResolvers.sample.acme.storage=/acme/certificates.json
#- --certificatesResolvers.sample.acme.httpChallenge.entryPoint=web
```

If you need more information about it, you can follow these links:

* [Traefik ACME](https://docs.traefik.io/https/acme/)
* [Traefik ACME Docker Compose](https://docs.traefik.io/user-guides/docker-compose/acme-tls/)

# DOCKER

## Install Docker and Docker-compose
If you have not installed docker and docker-compose, you can use the [following script](https://gist.githubusercontent.com/dascenciohz/b399436262e633a13a0344f2ad6b4359/raw/10910937fc8d11e1d8e20127d35d9ce15767eb5a/docker-install-ubuntu.sh) to perform both installations.

1. Download script

```
wget gist.githubusercontent.com/dascenciohz/b399436262e633a13a0344f2ad6b4359/raw/10910937fc8d11e1d8e20127d35d9ce15767eb5a/docker-install-ubuntu.sh
```

2. Permission file to execute

```
chmod +x docker-install-ubuntu.sh
```

3. Execute script

```
# Example to execute
./docker-install-ubuntu.sh [docker-compose-version] [myusername]

# execute
./docker-install-ubuntu.sh 1.25.0 piter
```

## Startup

1. Before starting the project we must configure the environment variables.

```
# Rename environtme file
mv env-example .env

# Edit environments
vim .env
```

2. To start the project, we execute:

```
docker-compose up --build -d
```

3. To recreate the project before any change, we execute:

```
docker-compose up --build -d --force-recreate
```

# WORKERS

By default Cronsun comes with a worker in PHP. If you want to add work nodes you can continue reading this document.

## How to add a custom node worker

To add a custom node we must create our own docker image. For example, if we want a work node that executes tasks programmed in Python we must create a Dockerfile similar to this:

```
## We use the python image we need
FROM python:3.5-alpine

LABEL maintainer="Daniel Ascencio <daniel.ascencio.hz@gmail.com>"

## Declare the environment variables
ENV GOROOT /usr/lib/go
ENV GOPATH /gopath
ENV GOBIN /gopath/bin
ENV PATH $PATH:$GOROOT/bin:$GOPATH/bin
ENV CRONSUN_VERSION 0.3.5

## Download and install cronsun
RUN set -x \
    && echo http://dl-cdn.alpinelinux.org/alpine/edge/community >> /etc/apk/repositories \
    && apk --no-cache add curl go git bash binutils-gold libc-dev \
    && mkdir -p /gopath/bin \
    && curl -L https://github.com/shunfei/cronsun/releases/download/v$CRONSUN_VERSION/cronsun-v$CRONSUN_VERSION-linux-amd64.zip -o cronsun.zip \
    && mkdir -p /opt \
    && unzip cronsun.zip -d /opt \
    && mv /opt/cronsun-v$CRONSUN_VERSION /opt/cronsun \
    && chmod +x /opt/cronsun/cronweb

EXPOSE 7079

WORKDIR /opt/cronsun

ENTRYPOINT []

## This line is important since we declare that it is a working node.
CMD ./cronnode -conf conf/base.json
```

From the web administrator we configure a new group called python and add the node.
Now you are ready to run jobs in python.

# CREDITS

- [shunfei/cronsun](https://github.com/shunfei/cronsun)