---
date: '2023-06-01'
lastmod: '2023-06-01'
draft: false
title: Basic Auth in Nginx
authors: ['Alex']
enableReadingTime: true
description: "Secure your website with Basic Authentication in Nginx. Learn how to set up .htpasswd, configure Nginx, and protect your web directories with a simple login prompt."
tags: ['auth', 'basic auth', 'devops', 'nginx']
categories: ['Development infrastructure']
featuredImage: '/posts/nginx-logo.png'
---

```
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
Install
```
sudo apt-get install apache2-utils
```
and create new password file and set password for user admin
```
htpasswd -c site.com.htpasswd admin
```

