# CRONSUN

**Cronsun** is a distributed cron-style job system. It's similar to crontab.

With Cronsun you can schedule your work to run in the background and at the intervals of time you want. You can schedule jobs in PHP, Node, Shell, MySQL and any other you want, you just have to configure a work node.

Cronsun uses [ETCD](https://github.com/etcd-io/etcd) in Cluster mode to store all the data corresponding to the scheduled jobs. Each output of the scheduled jobs is stored in [Mongo DB](https://www.mongodb.com/es).

# SECURITY

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

# CREDITS

- [shunfei/cronsun](https://github.com/shunfei/cronsun)
