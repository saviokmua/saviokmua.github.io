# How to start up docker compose on boot




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

