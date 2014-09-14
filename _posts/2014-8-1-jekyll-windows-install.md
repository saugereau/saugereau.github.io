---
layout: post
title: Install Jekyll on windows
category: Coding
tags: Jekyll
year: 2014
month: 8
day: 1
summary: Some guidelines to install ruby/pyhton and jekyll on windows OS
---

## Install Ruby

Go to [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)
You can use the rubyInstaller or 7-zip archive, but in this last case you have to add the ruby installation path to your PATH. Whatever your choice don't use folder that contains space.
To check your installation, open up a window command prompt

```
C:\Users\yourusername>ruby -v
ruby 2.0.0p481 (2014-05-08) [x64-mingw32]
```

RubyGems is a package manager which became part of the standard library in Ruby 1.9. It allows developers to search, install and build gems, among other features. All of this is done by using the gem command-line utility
A gem is a packaged Ruby application or library. It has a name (e.g. rake) and a version (e.g. 0.4.16)

```
C:\Users\yourusername>gem -v
2.0.14
```

Now, if you try to install jekyll, you get this warning

```
C:\Users\yourusername>gem install jekyll
Fetching: liquid-2.6.1.gem (100%)
Successfully installed liquid-2.6.1
Fetching: kramdown-1.4.1.gem (100%)
Successfully installed kramdown-1.4.1
Fetching: mercenary-0.3.4.gem (100%)
Successfully installed mercenary-0.3.4
Fetching: safe_yaml-1.0.3.gem (100%)
Successfully installed safe_yaml-1.0.3
Fetching: colorator-0.1.gem (100%)
Successfully installed colorator-0.1
Fetching: yajl-ruby-1.1.0.gem (100%)
ERROR:  Error installing jekyll:
        The 'yajl-ruby' native gem requires installed build tools.

Please update your PATH to include build tools or download the DevKit
from 'http://rubyinstaller.org/downloads' and follow the instructions
at 'http://github.com/oneclick/rubyinstaller/wiki/Development-Kit'

```

Go (again) on [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/) and download the development kit matching your ruby version. The package is a self-exatractable archive.

```
cd “C:\pathwhereyouextract”
C:\pathwhereyouextract>ruby dk.rb init
```

This initialize a config.yml in the devkit folder. This configuration file contains the absolute path locations of all installed Rubies to be enhanced to work with the DevKit.
To include any installed Rubies that were not automatically discovered, simply add a line.


Back to the Command Prompt, review (optional) and install.

```
C:\pathwhereyouextract>ruby dk.rb review
C:\pathwhereyouextract>ruby dk.rb install
[INFO] Updating convenience notice gem override for 'C:/Dev/ruby-2.0.0'
[INFO] Installing 'C:/Dev/ruby-2.0.0/lib/ruby/site_ruby/devkit.rb'
```

Now we can install jekyll

```
gem install jekyll
```

To create a new blog :

```
C:\Dev\Blog>jekyll new myblog
New jekyll site installed in C:/Dev/Blog/myblog.
```

If you try to run thius new website :

```
C:\Dev\Blog\myblog>jekyll serve
Configuration file: C:/Dev/Blog/myblog/_config.yml
            Source: C:/Dev/Blog/myblog
       Destination: C:/Dev/Blog/myblog/_site
      Generating...
  Liquid Exception: No such file or directory - python C:/Dev/ruby-2.0.0/lib/ru
y/gems/2.0.0/gems/pygments.rb-0.6.0/lib/pygments/mentos.py in _posts/2014-09-09
welcome-to-jekyll.markdown
                    done.
 Auto-regeneration: disabled. Use --watch to enable.
Configuration file: C:/Dev/Blog/myblog/_config.yml
    Server address: http://0.0.0.0:4000/
  Server running... press ctrl-c to stop.

```

There is an Liquid Exception, Liquid is the templating language to process templates used by Jekyll

## Install Python
Go to [https://www.python.org/download/](https://www.python.org/download/)
During the installation toggle "add python to classpath"

To verify installation, open a windows command prompt

```
C:\Users\yourusername>python --version
```

To add some new librairies to our python eco-system, we need to install EasyInstall.
EasyInstall is a package manager for Python that provides a standard format for distributing Python programs and libraries (based on the Python Eggs format). EasyInstall is a module bundled with Setuptools.[2] It is analogous to RubyGems for the Ruby.

For Windows 7 machines, download ez_setup.py and save it, for example, to C:\. Then run it using Python in a command prompt window:

```
C:\python ez_setup.py
```

It add some file to your python installation folder, for instance C:\Dev\Python-27\Scripts

Add C:\Dev\Python-27\Scripts to your Path

```
C:\Dev>easy_install --version
setuptools 5.7
```
Now we need to install pygments

```
C:\Dev>easy_install Pygments
```

This done you can start jekyll and this works

```
C:\Dev\Blog\myblog>jekyll serve
Auto-regeneration: disabled. Use --watch to enable.
onfiguration file: C:/Dev/Blog/myblog/_config.yml
   Server address: http://0.0.0.0:4000/
 Server running... press ctrl-c to stop.
```
