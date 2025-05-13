---
date: '2023-06-01'
lastmod: '2023-06-01'
draft: true
title: Roundcubemail in Docker
authors: ['Alex']
enableReadingTime: true
description: "Deploy Roundcube Webmail in Docker easily with this step-by-step guide. Set up your self-hosted webmail client with Docker Compose in minutes."
tags: ['roundcubemail', 'devops', 'docker']
categories: ['Development infrastructure']
featuredImage: '/posts/roundcube-logo.png'
---

docker-compose.yaml 
```
version: '2'

services:
  roundcubemail:
    image: roundcube/roundcubemail:latest
    restart: always
    volumes:
      - ./db/sqlite:/var/roundcube/db
      - /data/roundcubemail-docker/config.custom.inc.php:/var/roundcube/config/config.custom.inc.php
    ports:
      - 9002:80
    environment:
      - ROUNDCUBEMAIL_DB_TYPE=sqlite
      - ROUNDCUBEMAIL_SKIN=elastic
      - ROUNDCUBEMAIL_DEFAULT_PORT=993
      - ROUNDCUBEMAIL_DEFAULT_HOST=ssl://192.168.1.1
      - ROUNDCUBEMAIL_SMTP_SERVER=tls://192.168.1.1
```

config.custom.inc.php

```
<?php
  $config['db_dsnw'] = 'sqlite:////var/roundcube/db/sqlite.db?mode=0646';
  $config['db_dsnr'] = '';
  $config['imap_host'] = 'ssl://192.168.1.1:993';
  $config['smtp_host'] = 'tls://192.168.1.1:587';
  $config['temp_dir'] = '/tmp/roundcube-temp';
  $config['skin'] = 'elastic';
  $config['request_path'] = '/';
  $config['plugins'] = array_filter(array_unique(array_merge($config['plugins'], ['archive', 'zipdownload'])));
  $config['imap_conn_options'] = ['ssl' => ['verify_peer' => false, 'verify_peer_name' => false]];
  $config['smtp_conn_options'] = ['ssl' => ['verify_peer' => false, 'verify_peer_name' => false]];

```
