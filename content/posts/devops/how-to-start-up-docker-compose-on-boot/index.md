---
date: '2023-10-08'
lastmod: '2023-10-08'
draft: false
title: 'How to Start Docker Compose on Boot with systemd'
slug: 'how-to-start-up-docker-compose-on-boot'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to automatically start Docker Compose services on system boot using systemd. Step-by-step guide to create a systemd service for your Docker Compose project."
keywords: ['docker compose autostart', 'docker compose systemd service', 'docker compose on boot', 'linux docker startup', 'docker persistent service']
tags: ['docker', 'docker compose']
categories: ['DevOps']
hiddenFromHomePage: true
featuredImage: '/images/docker-compose.jpg'
---

# 🚀 How to Start Docker Compose on Boot Using systemd

Want your Docker containers to start automatically on boot? Here’s a clean way to run Docker Compose projects as services with `systemd`.

---

## 🛠 Example Project Structure

Suppose your project is located at:

```
/opt/myapp/docker-compose.yml
```

---

## 🧩 Step 1: Create a systemd service unit

Create a new file:

```
/etc/systemd/system/docker-compose-myapp.service
```

Paste the following content:

```
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

---

## 🚦 Step 2: Enable and start the service

Run the following commands:

```
sudo systemctl enable docker-compose-myapp
sudo systemctl start docker-compose-myapp
```

---

## ✅ Check service status

```
sudo systemctl status docker-compose-myapp
journalctl -u docker-compose-myapp
```

---

## 🧠 Notes

- Replace `/opt/myapp` with your actual project path.
- Make sure Docker and Docker Compose are properly installed.
- You can stop the service with:

```
sudo systemctl stop docker-compose-myapp
```

---