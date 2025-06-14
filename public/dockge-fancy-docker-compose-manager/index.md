# Dockge – Beautiful, Fancy, Easy-to-Use Docker Compose Manager


### Dockge – Beautiful, Fancy, Easy-to-Use Docker Compose Manager


Today I discovered a fantastic tool for managing my Docker containers based on docker-compose.yml files — it’s called [Dockge](https://dockge.kuma.pet/).

Dockge isn’t just another wrapper around Docker — it genuinely impressed me. It has a beautiful, modern interface, works fast, and most importantly, makes it super easy to start, stop, and monitor services in a stack. Some highlights:

![dockge](https://private-user-images.githubusercontent.com/1336778/280767691-26a583e1-ecb1-4a8d-aedf-76157d714ad7.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDk5MzU0OTUsIm5iZiI6MTc0OTkzNTE5NSwicGF0aCI6Ii8xMzM2Nzc4LzI4MDc2NzY5MS0yNmE1ODNlMS1lY2IxLTRhOGQtYWVkZi03NjE1N2Q3MTRhZDcucG5nP1gtQW16LUFsZ29yaXRobT1BV1M0LUhNQUMtU0hBMjU2JlgtQW16LUNyZWRlbnRpYWw9QUtJQVZDT0RZTFNBNTNQUUs0WkElMkYyMDI1MDYxNCUyRnVzLWVhc3QtMSUyRnMzJTJGYXdzNF9yZXF1ZXN0JlgtQW16LURhdGU9MjAyNTA2MTRUMjEwNjM1WiZYLUFtei1FeHBpcmVzPTMwMCZYLUFtei1TaWduYXR1cmU9ODkwYzMzYmVmZTVhODk1OGJiMDA5ZjY4MmVkNTQ5NTQ0M2JkMmIwN2YyMzUyMGJjMzBkNzExYWVjNzBkNmI3NCZYLUFtei1TaWduZWRIZWFkZXJzPWhvc3QifQ.GNoMsRhoZ9b6JrjlA0ZG57YgGIvOc0z3XdaYATEr3Dk)

- Fully self-hosted — I just added it as a service in my existing docker-compose.yml.

- Supports multiple stacks and lets you switch between them effortlessly.
- 
- Displays logs, statuses, ports, and even lets you edit files directly from the UI.

It’s exactly what I’ve been missing for managing several projects — each with their own setup of Redis, Postgres, Sidekiq, web servers, and more.

If you’re managing multiple containers and tired of running docker compose ps, logs, up -d all the time in the terminal — give Dockge a try. It might just become your new favorite DevOps tool.

[Official web page](https://dockge.kuma.pet/)

[Github](https://github.com/louislam/dockge)

### My custom config

```dockerfile
services:
  dockge:
    image: louislam/dockge:1
    restart: unless-stopped
    ports:
      # Host Port : Container Port
      - 5001:5001
    volumes:
      - /Users/savio/.docker/run/docker.sock:/var/run/docker.sock
      - /Users/savio/#_Docker/dockge/data:/app/data
        
      # If you want to use private registries, you need to share the auth file with Dockge:
      # - /root/.docker/:/root/.docker

      # Stacks Directory
      # ⚠️ READ IT CAREFULLY. If you did it wrong, your data could end up writing into a WRONG PATH.
      # ⚠️ 1. FULL path only. No relative path (MUST)
      # ⚠️ 2. Left Stacks Path === Right Stacks Path (MUST)
      - /Users/savio/#_Docker/dockge/stacks:/opt/stacks
    networks:
     - shared
    environment:
      # Tell Dockge where is your stacks directory
      - DOCKGE_STACKS_DIR=/opt/stacks
networks:
 shared:
  external: true
```
