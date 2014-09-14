---
layout: post
title: Scala, Play configuration tips
category: Coding
tags: scala play
year: 2014
month: 8
day: 1
summary: Ivy configuration, IDE plugins ...
---


### SBT : change local Ivy repository path

Add to your sbt/conf/sbtconfig.txt file : 
-Dsbt.ivy.home="\yourivylocalfolder"

### SBT plugins to create project definitions
Add the folllowing lines in your %userprofile%\.sbt\plugins\build.sbt

For [Eclipse](https://github.com/typesafehub/sbteclipse) : addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.2.0")
For [IntelliJ](https://github.com/mpeltonen/sbt-idea) : addSbtPlugin("com.github.mpeltonen" % "sbt-idea" % "1.6.0")


### Activator : change local Ivy repository path

Add the following line in activator.bat : 
set SBT_OPTS=-Dsbt.ivy.home="D:\Fichier Development\Ivy-repo"

