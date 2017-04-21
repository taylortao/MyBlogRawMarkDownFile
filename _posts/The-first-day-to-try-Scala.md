---
title: The first day to try Scala and Play framework
date: 2017-04-20 15:10:11
categories:
 - IT Technology
tags:
 - Scala
 - Play
---

### Install scala
1. Download scala-X.XX.tgz from the official site.
2. Unzip archive and move it to /usr/local folder
If you step into ./bin folder, you could use `./scala` to run it.
3. Open your .bash_profile file :
`sudo nano ~/.bash_profile`
4. Add scala/bin folder to your PATH
`export PATH=/usr/local/scala/bin:$PATH`
Control + X to save and restart.
5. Check successful by typing Scala in terminal
If you see below, then Scala is successfully installed.
```
Welcome to Scala 2.12.2 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_121).
Type in expressions for evaluation. Or try :help.
```

<!-- more -->

### Play framework
1. Download a project from [Play Framework](https://playframework.com/download).
2. Run `./sbt run`
It will automatically download all the necessaries during the first time.
3. Navigate to `localhost:9000`, if you see a web page, it is successful.

