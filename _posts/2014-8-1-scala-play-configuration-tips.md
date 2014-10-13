---
layout: post
title: Scala, Play configuration tips
category: Coding Tips
tags: scala play
summary: Ivy configuration, IDE plugins ...
---


### SBT : change local Ivy repository path

Add to your sbt/conf/sbtconfig.txt file : 
-Dsbt.ivy.home="\yourivylocalfolder"

### SBT plugins to create project definitions
Add the folllowing lines in your %userprofile%\.sbt\plugins\build.sbt

- For [Eclipse](https://github.com/typesafehub/sbteclipse) : addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.2.0")
- For [IntelliJ](https://github.com/mpeltonen/sbt-idea) : addSbtPlugin("com.github.mpeltonen" % "sbt-idea" % "1.6.0")


### Activator : change local Ivy repository path

Add the following line in activator.bat : 
set SBT_OPTS=-Dsbt.ivy.home="D:\Fichier Development\Ivy-repo"

### Ide plugins

#### Eclipse

The url for update site are on [Scala-Ide web site](http://scala-ide.org/download/current.html), there is one plugin per scala version, so the plugin for scala 2.10 will not work for project in 2.11. And you can't have the two plugin version on the same eclipse installation.

#### Intellij
The [JetBrain scala plugin](http://confluence.jetbrains.com/display/SCA/Scala+Plugin+for+IntelliJ+IDEA) already include SBT plugin, so you don' have to add another SBT plugin (there is few plugin SBT if you search in plugin repository)
Once JetBrain Scala plugin installed, you could define a custom sbt launcher :

<figure>
  <img src="/blog/assets/images/scala-play-configuration-tips/sbt-config.png" />
  <figcaption>SBT configuration screen</figcaption>
</figure> 

But this doesn't allow to parameter the ivy home, and when you open a sbt project for the first time without (iml file), inteelij try to import all dependancies and copy them in the default ivy repository (for instance C:\Users\yourusername\.ivy2\cache).
An another way is to update VM parameters
 
 <figure>
   <img src="/blog/assets/images/scala-play-configuration-tips/sbt-jvm-parameters.png" />
   <figcaption>Jvm parameters</figcaption>
 </figure> 