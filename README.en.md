# docker-wordpress
Docker environment stack for Wordpress with PHP-apache, MariaDB and MailHog for developments.


## Overview

Requirement **docker-compose** 


### Stack Setup

* Wordpress - PHP-apache
* MariaDB
* MailHog


### Docker Images

* wordpress:${WP_CORE_VERSION}-php${PHP_VERSION}-apache
* mariadb:10.3
* mailhog/mailhog


### Reference

* https://hub.docker.com/_/wordpress
* https://hub.docker.com/_/mariadb
* https://hub.docker.com/r/mailhog/mailhog/
* https://hub.docker.com/_/php


### Project Tree

```
docker-wordpress
│   .env
│   .env.example
│   .gitignore
│   docker-compose.yml
│   LICENSE
│   README.md
│       
├───db.backup
│   └───.gitignore
│
├───db.init
│   └───zzz-migrate.sql.example
│
├───wp
│   ├───wp-admin
│   ├───wp-includes
│   ├───wp-content (empty)
│   └───wp-*.php
│
└───wp-content
    ├───mu-plugins
    ├───plugins
    └───themes
```


### Environment Variables

#### Environment file: `.env`

_`.env` file file is essential! Use `.env.example` as template._

**Rename `.env.example` as `.env` or create a copy renamed `.env`**

##### Variable description

* `PROJECT_NAME`: Name of your docker stack.
* `VERSION_ID`: ID of your project stack. (Use it only if you want try more version of your project).
* `WEB_PORT_EXPOSED`: HTTP port exposed for your site _(default 80)_.
* `PHP_VERSION`: The PHP version you want to use _(default 7.2)_.
* `DB_PORT_EXPOSED`: Used only if you want connect to database with external client _(HeidiSQL_, _MySQL Workbench, etc.)_. Client configuration _host_ `127.0.0.1`, _username_ `root` whereas _password_ your `DB_ROOTPASS` and _port_ your `DB_PORT_EXPOSED` as in environment file _(default 3306)_.
* `DB_ROOTPASS`: MySQL root password. It's not safe use this method in production but for development it's acceptable.
* `DB_USER`: User name for database connection.
* `DB_PASS`: User password for database connection.
* `DB_NAME`: Database name of your wordpress site. Will be created at the build. Otherwise you can create database from docker exec or external client.
* `WP_CORE_VERSION`: The Wordpress version you want to use.
* `WP_TABLE_PREFIX`: Database table prefix of your wordpress site.


### Docker

Command in nutshell for build, start, stop, remove and validate the docker stack.

* `docker-compose up -d --build` - builds and init your stack
* `docker-compose down` - remove your stack but keep your database and source
* `docker-compose start` - start your stack
* `docker-compose stop`  - stop your stack 
* `docker-compose down --volumes` - like _down_ but also removes volumes (delete database and core)
* `docker-compose config` - validate your stack configuration

* `docker-compose exec SERVICE_NAME bash` enters with bash terminal into container
  * SERVICE_NAME = `wp` for enter in web server
  * SERVICE_NAME = `db` for enter in database server
  * SERVICE_NAME = `mailhog` for enter in smtp server
