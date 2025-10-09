---
date: '2025-10-09'
lastmod: '2025-10-09'
draft: false
title: "How to Set Up a Private Docker Registry with UI and Authentication"
slug: 'how-to-setup-private-docker-registry-with-ui-and-auth'
authors: ['Alex']
enableReadingTime: true
description: "Step-by-step guide to setting up a private Docker Registry with web UI and basic authentication via Nginx"
keywords: ["docker registry", "private docker registry", "docker registry ui", "docker registry nginx", "htpasswd docker", "joxit registry ui", "self-hosted registry"]
tags: ['docker', 'registry', 'nginx', 'devops', 'self-hosted']
categories: ['DevOps']
hiddenFromHomePage: false
---

### How to Set Up a Private Docker Registry with UI and Authentication

If you work with Docker and need your own private registry to store images — this guide is for you.
Let's explore how to deploy Docker Registry with a convenient web interface and set up authentication through Nginx.

## 🔧 Docker Compose Configuration

First, let's create a `docker-compose.yml` file with two services: the registry itself and a UI for it.

```yaml
version: '3'
services:
  registry:
    image: registry:2
    ports:
      - "5005:5000"
    volumes:
      - ./data:/var/lib/registry
      - ./auth:/auth
    environment:
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      # Enable authentication
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    restart: unless-stopped
  registry-ui:
    image: joxit/docker-registry-ui:latest
    ports:
      - "8089:80"
    environment:
      REGISTRY_TITLE: My Docker Registry
      # UI connects to registry locally
      NGINX_PROXY_PASS_URL: http://registry:5000
      SINGLE_REGISTRY: "true"
      DELETE_IMAGES: "true"
      SHOW_CATALOG_NB_TAGS: "true"
      # UI will pass credentials
      REGISTRY_SECURED: "false"
    depends_on:
      - registry
    restart: unless-stopped
```

**Key points explained:**

- **registry:2** — official Docker Registry version 2 image
- **REGISTRY_STORAGE_DELETE_ENABLED** — allows deleting images via API
- **REGISTRY_AUTH: htpasswd** — use basic authentication via htpasswd
- **joxit/docker-registry-ui** — convenient web interface for viewing and managing images
- **DELETE_IMAGES: "true"** — allows deleting images through UI

## 🔐 Creating Password File

For authentication, you need to create an htpasswd file. First, install the utilities:

```bash
sudo apt-get install apache2-utils
```

Create a directory for auth files and generate htpasswd:

```bash
mkdir -p auth
htpasswd -Bc auth/htpasswd username
```

> [!NOTE]
> The `-B` flag uses bcrypt for encryption (recommended), `-c` creates a new file

## 🌐 Nginx Configuration

To access the registry through a single domain, let's configure Nginx as a reverse proxy:

```nginx
# UI on root
location / {
    proxy_pass http://10.22.1.117:8089;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# Registry API on /v2/
location /v2/ {
    proxy_pass http://10.22.1.117:5005;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Pass Authorization header to registry
    proxy_set_header Authorization $http_authorization;
    proxy_pass_header Authorization;

    client_max_body_size 0;
    chunked_transfer_encoding on;
}
```

**Important details:**

- **location /** — proxies UI to the domain root
- **location /v2/** — proxies registry API (all Docker commands work through /v2/)
- **proxy_set_header Authorization** — passes authentication from client to registry
- **client_max_body_size 0** — allows uploading images of any size
- **chunked_transfer_encoding on** — required for correct transfer of large files

## 🚀 Deployment and Usage

Start the services:

```bash
docker-compose up -d
```

Now you can work with the registry:

```bash
# Login to registry
docker login registry.yourdomain.com

# Push an image
docker tag myimage:latest registry.yourdomain.com/myimage:latest
docker push registry.yourdomain.com/myimage:latest

# Pull an image
docker pull registry.yourdomain.com/myimage:latest
```

The web interface will be available at `https://registry.yourdomain.com`,
and the Docker API at `https://registry.yourdomain.com/v2/`

## ✅ Security Recommendations

- **Use HTTPS** — make sure to configure an SSL certificate (Let's Encrypt)
- **Restrict access** — add IP whitelisting or VPN for additional protection
- **Regular backups** — keep the `./data` folder with images backed up
- **Disk space monitoring** — images quickly consume space, configure cleanup of old tags

---

Now you have your own private Docker Registry with a convenient interface! 🎉