---
title: install mysql under mac
date: 2017-04-22 18:47:02
categories:
 - IT Technology
tags:
 - MySQL
toc: false
---

1. Install HomeBrew
```
ruby -e "$(curl --insecure -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

2. `brew install mysql`
3. Start mysql service: `mysql.server start`
4. Set password: `mysql_secure_installation`
5. Login: `mysql -u root -p`
