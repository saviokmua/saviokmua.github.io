---
date: '2023-10-08'
lastmod: '2023-10-08'
draft: false
title: 'How to start up docker compose on boot'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to configure an NFS server and client for file sharing in Linux. Step-by-step guide to set up NFS server, export directories, and mount them on client machines."
tags: ['NFS', 'linux', 'ubuntu', 'server', 'client']
categories: ['Development infrastructure']
featuredImage: 'nfs-logo.png'
---



-
For example, we have `/opt/myapp/docker-compose.yml`

1) Create `/etc/systemd/system/docker-compose-myapp.service`
```bash
[Unit]
Description=Docker Compose Application Service
Requires=docker.service
After=docker.service
[Service]
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0
Restart=on-failure
StartLimitIntervalSec=60
StartLimitBurst=3
[Install]
WantedBy=multi-user.target
```

```
systemctl enable docker-compose-app
systemctl start docker-compose-app
```
