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
├───wp-content
    ├───mu-plugins
    ├───plugins
    └───themes
```
