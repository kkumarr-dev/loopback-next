---
lang: en
title: 'Setting up MySQL in Docker'
keywords: LoopBack 4.0, LoopBack 4, Node.js, TypeScript, OpenAPI, Tutorial
sidebar: lb4_sidebar
permalink: /doc/en/lb4/todo-list-tutorial-docker-mysql.html
summary: LoopBack 4 TodoList Application Tutorial - Setting up MySQL in Docker
---

Here are the instructions for getting MySQL running in a docker container for
the purposes of running the TodoList example against a relational database.

## Install Docker

For downloading and installing docker, see
[Get Docker](https://docs.docker.com/get-docker/) for instructions on how to
install it on your respective operating system.

## Get MySQL docker image

```sh
docker pull mysql/mysql-server:latest
```

## List MySQL docker image

```sh
docker images
```

outputs:

```sh
REPOSITORY TAG IMAGE ID CREATED SIZE
mysql/mysql-server latest 8a3a24ad33be 2 months ago 366MB
```

## Create a docker container based on the image, and set root password and port

We will give the docker container a name of `todo-mysql` and we will give the
`root` user a password of `my-secret-pw`.

```sh
docker run --name todo-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw  -d mysql/mysql-server
```

outputs a UUID for the container:

```sh
191508fcbcfd3b683fea06bb3c8a2f1f252527a15891b58b124518cf2cb60249
```

If the operating system complains that your 3306 port is already being used on
your machine, you can remove the container, and try a different port number.

```sh
docker rm todo-mysql
docker run --name todo-mysql -p 3307:3306 -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql/mysql-server
```

## View the status of your docker container on your machine

```sh
docker ps

OR

docker ps -a
```

The first lists running containers, and the second lists ALL containers (running
and stopped).

If it's up and running, the output should look like this:

```sh
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS                   PORTS                               NAMES
24505c80a83d        mysql/mysql-server   "/entrypoint.sh mysq…"   7 minutes ago       Up 7 minutes (healthy)   0.0.0.0:3306->3306/tcp, 33060/tcp   todo-mysql
```

## Fixing the root user info in the database

Recent versions of MySQL use
[SHA-2 Pluggable Authentication](https://dev.mysql.com/doc/refman/8.0/en/caching-sha2-pluggable-authentication.html).

A lot of Node.js application are accustomed to the previous authentication
method in MySQL named
[Native Pluggable Authentication](https://dev.mysql.com/doc/refman/8.0/en/native-pluggable-authentication.html).

For the purposes of this TodoList example application, we need to have MySQL
switch to the `native` authentication method.

In order to accomplish this, we need to delete the root user's definition, and
create the root user again but with a different definition.

Open up a MySQL interactive prompt into the container

```sh
docker exec -it todo-mysql mysql -uroot -p
```

This opens the `mysql` interactive prompt. You will be prompted for the
password. Enter `my-secret-pw`.

Perform the following commands:

```sh
show databases;
use mysql;
SELECT user,host FROM user;
```

You should see the user `'root'@'localhost'` listed in the results

Continue with the following commands:

```sh
drop user 'root'@'localhost';
CREATE USER 'root'@'%' IDENTIFIED BY 'my-secret-pw';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

Type `exit` to get out of the interactive shell

Now stop and start the container.

```sh
docker stop todo-mysql
docker start todo-mysql
```

We need to go in again and change one more thing (we are not able to do it in
the same mysql session.)

Open the mysql prompt again:

```sh
docker exec -it todo-mysql mysql -uroot -p
```

You will be prompted for the password. Enter `my-secret-pw`.

Perform the following commands:

```sh
use mysql;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'my-secret-pw';
FLUSH PRIVILEGES;
```

Type `exit` to get out of the interactive shell

Now stop and start the container.

```sh
docker stop todo-mysql
docker start todo-mysql
```

Check the logs of the container to see if everything is ok.

```sh
docker logs todo-mysql
```

You should see output like this:

```sh
[Entrypoint] MySQL Docker Image 8.0.21-1.1.17
[Entrypoint] Initializing database
2020-10-10T20:08:22.809376Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.21) initializing of server in progress as process 20
2020-10-10T20:08:22.823401Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-10-10T20:08:39.810201Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-10-10T20:09:12.821296Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
[Entrypoint] Database initialized
2020-10-10T20:10:23.747797Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.21) starting as process 81
2020-10-10T20:10:24.523215Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-10-10T20:10:26.282045Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-10-10T20:10:27.294389Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
2020-10-10T20:10:28.316478Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-10-10T20:10:28.317986Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2020-10-10T20:10:28.436967Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.21'  socket: '/var/lib/mysql/mysql.sock'  port: 0  MySQL Community Server - GPL.
Warning: Unable to load '/usr/share/zoneinfo/iso3166.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/leapseconds' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/tzdata.zi' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone.tab' as time zone. Skipping it.
Warning: Unable to load '/usr/share/zoneinfo/zone1970.tab' as time zone. Skipping it.

[Entrypoint] ignoring /docker-entrypoint-initdb.d/*

2020-10-10T20:10:35.615148Z 10 [System] [MY-013172] [Server] Received SHUTDOWN from user root. Shutting down mysqld (Version: 8.0.21).
2020-10-10T20:11:10.605086Z 0 [System] [MY-010910] [Server] /usr/sbin/mysqld: Shutdown complete (mysqld 8.0.21)  MySQL Community Server - GPL.
[Entrypoint] Server shut down

[Entrypoint] MySQL init process done. Ready for start up.

[Entrypoint] Starting MySQL 8.0.21-1.1.17
2020-10-10T20:11:11.060362Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.21) starting as process 151
2020-10-10T20:11:11.515963Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-10-10T20:11:14.533342Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-10-10T20:11:14.833765Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2020-10-10T20:11:15.385223Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-10-10T20:11:15.385694Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2020-10-10T20:11:15.482368Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.21'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server
- GPL.
[Entrypoint] MySQL Docker Image 8.0.21-1.1.17
[Entrypoint] Starting MySQL 8.0.21-1.1.17
2020-10-10T20:47:36.825667Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.21) starting as process 22
2020-10-10T20:47:36.871574Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2020-10-10T20:47:38.685523Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2020-10-10T20:47:38.954109Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2020-10-10T20:47:39.109335Z 0 [System] [MY-010229] [Server] Starting XA crash recovery...
2020-10-10T20:47:39.121060Z 0 [System] [MY-010232] [Server] XA crash recovery finished.
2020-10-10T20:47:40.488010Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2020-10-10T20:47:40.488446Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2020-10-10T20:47:40.663988Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.21'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server
- GPL.
```

Looks like everything is OK.

## IP Address of the MySQL Docker container

For Mac and Linux users, the IP address of the docker container running on your
machine will be the IP address of your machine : `127.0.0.1` .

For Windows users, it isn't, and you can find the proper IP address by issuing
the command

```sh
docker-machine ip default
```

You need to know the proper IP address and port values of your MySQL database
when you set the `host` and `port` values in your `db.datasource.ts` file.
[Specifying the datasource properties](./todo-list-tutorial-sqldb.md#specifying-the-datasource-properties).
