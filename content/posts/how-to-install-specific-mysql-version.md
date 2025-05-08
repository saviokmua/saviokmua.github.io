+++
date = '2025-05-08T15:01:30+03:00'
draft = true
title = 'How to Install Specific Mysql Version'
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
