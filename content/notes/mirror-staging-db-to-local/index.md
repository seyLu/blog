---
title: Mirror Staging Database to Local Database
description: "MySQL and Docker edition"
date: "2024-06-11"
categories: ["Notes"]
tags: ["docker", "mysql", "sql", "bash", "debugging"]
image: Murasaki_shikibu_php_mysql_book.png
---

## Staging db dump

`mysqldump -u username -p dbname > local_dump.sql`

## Copy local file to docker container

`docker cp local_dump.sql container_name:/path/in/container/local_dump.sql
`

## Exec MYSQL container

`docker exec -it container_name /bin/bash`

## Create a new database

`mysql> CREATE DATABASE dbname;`

## Import dump to local

`mysql -u username -p dbname < /path/in/container/local_dump.sql`

-   Rebuild Container for sanity
-   Change db to use in web server (most prolly in a config file somewhere)
