---
title: "How to Change Grafana Admin Password"
slug: "how-to-change-grafana-admin-password"
date: "2023-06-01"
lastmod: "2023-06-01"
draft: false
authors: ["Alex"]
description: "Learn how to reset your Grafana admin password using grafana-cli. Simple step-by-step CLI guide for login recovery and admin access."
keywords: ["grafana", "admin password reset", "grafana-cli", "grafana security", "grafana login", "recover grafana admin", "devops tools"]
tags: ["grafana", "devops", "password"]
categories: ["DevOps"]
hiddenFromHomePage: true
enableReadingTime: true
featuredImage: "how-to-change-grafana-password.png"
---

# 🔐 How to Change Grafana Admin Password

If you've forgotten the admin password in Grafana or just want to reset it for security reasons, there's a simple command-line tool you can use to reset it instantly. This guide walks you through changing the **Grafana administrator password** using the built-in CLI.

---

## ⚡ Quick Command

```
bash
grafana-cli admin reset-admin-password <new_password>
```

Replace `<new_password>` with your desired secure password.

> 🛑 You must run this command as the same user that runs the Grafana service (usually: `grafana`).

---

## 🛠️ Step-by-Step Instructions

1. **SSH into your server** or open a terminal if local.
2. *(Optional)* Stop Grafana:
   ```
   bash
   sudo systemctl stop grafana-server
   ```
3. Run the CLI password reset:
   ```
   bash
   grafana-cli admin reset-admin-password mySecurePassword123
   ```
4. Start Grafana again:
   ```
   bash
   sudo systemctl start grafana-server
   ```
5. Log in to `http://localhost:3000` with:
    - **Username:** `admin`
    - **Password:** `mySecurePassword123`

---

## ✅ When to Use This

- Lost or forgot admin password
- After a fresh install of Grafana
- After breaking login due to failed SSO/SAML setup
- Server or container migration


