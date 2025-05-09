+++
date = '2023-10-13T15:01:30+03:00'
lastmod = '2023-10-13T15:01:30+03:00'
draft = false
title = 'How to fix incorrect locale'
authors = ['Alex']
enableReadingTime = true
description = "Learn how to fix incorrect locale settings on Linux and other systems. Step-by-step instructions to resolve locale-related issues, ensure proper language settings, and improve system performance."
tags = ['locale', 'linux', 'ubuntu']
categories = ['Development infrastructure']
featuredImage = 'mysql.svg'
+++

# Can't set locale; make sure $LC_* and $LANG are correct!

```bash
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
just run
```bash
sudo dpkg-reconfigure locales
```
