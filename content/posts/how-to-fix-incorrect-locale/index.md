---
date: '2023-10-13'
lastmod: '2023-10-13'
draft: false
title: 'How to Fix Incorrect Locale Warnings on Linux'
slug: 'how-to-fix-incorrect-locale-warnings-on-linux'
authors: ['Alex']
enableReadingTime: true
description: "Learn how to fix incorrect locale settings on Linux. Step-by-step instructions to resolve locale-related issues, avoid warning messages, and ensure correct system language configuration."
keywords: ['linux locale fix', 'dpkg-reconfigure', 'LC_ALL', 'LANG', 'perl locale warning', 'ubuntu locale', 'can’t set locale', 'locales configuration']
tags: ['locale', 'linux', 'ubuntu']
categories: ['DevOps']
hiddenFromHomePage: true
featuredImage: 'how-to-fix-incorrect-locale.jpg'
---

# 🌍 How to Fix Incorrect Locale Warnings on Linux

Are you getting locale-related warnings on your Linux system such as:

```
bash
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
LANGUAGE = "en_US:en",
LC_ALL = (unset),
LC_TIME = "uk_UA.UTF-8",
LC_MONETARY = "uk_UA.UTF-8",
LC_ADDRESS = "uk_UA.UTF-8",
LC_TELEPHONE = "uk_UA.UTF-8",
LC_NAME = "uk_UA.UTF-8",
LC_MEASUREMENT = "uk_UA.UTF-8",
LC_IDENTIFICATION = "uk_UA.UTF-8",
LC_NUMERIC = "uk_UA.UTF-8",
LC_PAPER = "uk_UA.UTF-8",
LANG = "en_US.UTF-8"
are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
apt-listchanges: Can't set locale; make sure $LC_* and $LANG are correct!
```

These warnings typically appear when the system is missing the required locale definitions. Here's how to fix it.

---

## ✅ Solution: Reconfigure Locales

To generate and configure the appropriate locales on Debian-based systems (e.g., Ubuntu), run:

```bash
sudo dpkg-reconfigure locales
```

### 📝 Steps:

1. Select one or more locales (use space to mark), e.g., `en_US.UTF-8` and/or `uk_UA.UTF-8`.
2. Choose the default locale when prompted.
3. Confirm and wait for the system to regenerate the locale files.

---

## 💡 Alternative: Manually Set Locales (Optional)

For Docker containers, CI/CD, or minimal systems, you can set locale variables manually:

```bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

To make this persistent, add the above lines to your shell profile (`~/.bashrc`, `~/.zshrc`, or `/etc/environment`).