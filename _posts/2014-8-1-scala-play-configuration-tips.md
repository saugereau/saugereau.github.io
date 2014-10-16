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
 
If you want to import an existing scala project for the first time as sbt module, the scala version of your module is defined by sbt, so the information in the Project Structure > Modules Facet Scala display the correct compiler.
For instance with the akka-spray-websocket template gtom [typesafe site](http://typesafe.com/activator/template/akka-spray-websocket) wtih this sbt file :

```

version       := "0.2"
scalaVersion  := "2.10.3"
scalacOptions := Seq("-unchecked", "-deprecation", "-encoding", "utf8")
resolvers ++= Seq(
  "Typesafe repository" at "http://repo.typesafe.com/typesafe/releases/",
  "Spray repository" at "http://repo.spray.io/"
)

libraryDependencies ++= {
  val akkaV = "2.2.3"
  val sprayV = "1.2.0"
  Seq(
//  "org.java-websocket"  %   "Java-WebSocket" % "1.3.1",
    "io.spray"            %%  "spray-json"     % "1.2.5",
    "io.spray"            %   "spray-can"      % sprayV,
    "io.spray"            %   "spray-routing"  % sprayV,
    "com.typesafe.akka"   %%  "akka-actor"     % akkaV,
    "com.typesafe.akka"   %%  "akka-testkit"   % akkaV   % "test",
    "io.spray"            %   "spray-testkit"  % sprayV  % "test",
    "org.scalatest"       %%  "scalatest"      % "2.0"   % "test",
    "junit"               %   "junit"          % "4.11"  % "test",
    "org.specs2"          %%  "specs2"         % "2.2.3" % "test"
  )
}

seq(Revolver.settings: _*)

```


 <figure>
   <img src="/blog/assets/images/scala-play-configuration-tips/project-import.png" />
   <figcaption>SBT project import</figcaption>
 </figure> 
 
 Now if you watch the Project Structure screen :
 
 <figure>
   <img src="/blog/assets/images/scala-play-configuration-tips/project-structure.png" />
   <figcaption>Project structure - scala facet</figcaption>
 </figure> 
 
 
 When you create a new project you could define your scala librairies :
 
 <figure>
   <img src="/blog/assets/images/scala-play-configuration-tips/new-scala-module.png" />
   <figcaption>Project structure - scala facet</figcaption>
 </figure> 
 
 By default, it takes the value of your SCALA_HOME environment variable.
 Don't forget to change (eventually the "compiler library" and "standard library" value if you need to allow multiple value and check "Make global librairies" to keep this settings for others projects.