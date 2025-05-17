---
date: '2023-06-01'
lastmod: '2023-06-01'
draft: false
title: Basic Authentication in Nginx — Protect Your Website with a Password
slug: 'basic-authentication-in-nginx-protect-your-website-with-a-password'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to set up Basic Authentication in Nginx to password-protect parts of your website using .htpasswd. Step-by-step guide with examples and security tips."
keywords: ["Nginx basic auth", "nginx password protect", "htpasswd nginx", "protect admin area nginx", "restrict access nginx", "nginx user authentication"]
tags: ['auth', 'basic auth', 'nginx']
categories: ['DevOps']
hiddenFromHomePage: true
featuredImage: 'basic-auth-nginx.webp'
---

Sometimes you need to restrict access to specific areas of your website—such as an admin panel, 
staging environment, or internal documentation. One of the simplest ways to do this is by enabling 
**Basic Authentication** using **Nginx**. In this guide, you’ll learn how to set up password protection in just a few steps.


## 🔧 Nginx Configuration Example with Basic Auth
```config
server {
    server_name site.com;
    location / {
        ...
	      auth_basic           "Administrator Area";
        auth_basic_user_file /etc/nginx/protections/site.com.htpasswd;
        ...
    }
}
```

- `auth_basic` sets the authentication prompt message (realm).

- `auth_basic_user_file` specifies the path to the password file.


## 🛠 Installing Utilities & Creating a Password File

First, install the tool that allows you to create .htpasswd files:
```bash
sudo apt-get install apache2-utils
```
Now create a new password file and add the admin user:
```bash
htpasswd -c site.com.htpasswd admin
```

> [!NOTE]
> The `-c` flag creates a new file, overwriting it if it exists. Don't use `-c` when adding additional users.

Move the file to your desired location, such as:
```bash
sudo mv site.com.htpasswd /etc/nginx/protections/
```
<br>
Make sure to test your Nginx configuration and reload it:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## ✅ Best Practices for Security

- Store the .htpasswd file outside the publicly accessible directories.

- For more robust authentication, consider using OAuth, JWT, or integration with a centralized identity provider.

- Don’t forget to enforce HTTPS to protect credentials during transmission.