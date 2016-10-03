---
layout: post
title:  "Building ReactOS under GNU/Linux"
date:   2016-09-24 21:09:32 -0300
categories: React OS
---

[React OS](http://www.reactos.org/) is a free-software/open source clone of MS-Windows, in active development. I am a free software advocate, so I want to help to test it, even though I am a GNU/Linux user/programmer, and I know nothing about the MS-Windows internals... In this blog, I will share some experiences about it, so you can also join the React OS testing/development.

My first step was to build React OS from the sources. In order to do so, the first thing that you need to do is to build your React OS build environment, which provides versions of binutils/gcc etc. specially prepared for cross-compiling React OS under GNU/Linux.


1) Create a directory to work in. For example

{% highlight bash %}
mkdir reactos-work

cd reactos-work
{% endhighlight %}

2) Get the current version of the React OS build environment from the React OS page at sourceforge, for example

<http://sourceforge.net/projects/reactos/files/RosBE-Unix/2.1.2/RosBE-Unix-2.1.2.tar.bz2/>

and extract it to your working directory, with the order

{% highlight bash %}
tar xvf RosBE-Unix-2.1.1.tar.bz2
{% endhighlight %}

3) Run the script RosBE-builder.sh telling it where do you want to create your React OS build environment.  Even tough the script recommends that you run it as root, don't do it! It is not really need it, and its always a bad idea to run something as root when it is not the case.

{% highlight bash %}
./RosBE-Builder.sh /home/user/reactos-work/RosBE
{% endhighlight %}

(change here /home/user/reactos-work by the path where you have installed
RosBE !)

It will check for the necessary dependencies (bison, flex, gcc,g++, grep, makeinfo and GNU make), 
Should any of them be missing, install it.If something goes wrong, the script will tell you and you might want to check the 
build log. In that case, report it in Reactos
[Jira](http://jira.reactos.org/).


4) OK, if everything went fine, now you are ready to build React OS from the sources.To enter the React OS build environment, run

{% highlight bash %}
/home/user/reactos-work/reactos-work/RosBE/RosBE.sh
{% endhighlight %}

You should see something like

{% highlight bash %}
*******************************************************************************

*         ReactOS Build Environment for Unix-based Operating Systems          *

*                      by Colin Finck <colin@reactos.org>                     *

*                                                                             *

*                                 Version 2.1.2                               *

*******************************************************************************


For a list of all included commands, type: "help"
{% endhighlight %}


5) Now get the current version of the sources using subversion. Use 

{% highlight bash %}
svn checkout svn://svn.reactos.org/reactos/trunk/reactos
{% endhighlight %}

if you just want the main React OS source code, o 

{% highlight bash %}
svn checkout svn://svn.reactos.org/reactos/trunk/
{% endhighlight %}

if you want some extra stuff (documentation,rostests, wallpapers, etc.)

React OS is currently using cmake and ninja (which are provided in the React OS development environment) for the building process. 

6) Finally, create a directory rosbuild for your build, and run the configure 
script from the top of the source tree:

{% highlight bash %}
mkdir rosbuild

cd rosbuild

../reactos/configure.sh
{% endhighlight %}

Finally, run ninja bootcd inside the rosbuild directory for creating the boot cd for React OS.

{% highlight bash %}
ninja bootcd
{% endhighlight %}

This will create a file named bootcd.iso in the top of the source tree, wich is the iso image of the boot cd. Alternatively you can use the command "ninja livecd" for creating the live cd.



When you test React OS, it is useful to make a clean build of the current sources. You may want therefore to automate this process using a shell script like this:

{% highlight bash %}
#!/bin/bash -v
DONDE=/home/user/reactos-work
rm --recursive --force rosbuild
mkdir rosbuild
cd $DONDE/rosbuild
$DONDE/trunk/reactos/configure.sh
cd $DONDE/rosbuild/host-tools
ninja all
cd $DONDE/rosbuild/reactos
ninja bootcd
{% endhighlight %}

Also it is useful to tag the boot cd image, with the number of the current revision (since when doing testing you will end up with many different iso images!). I am using a python script for doing that:

{% highlight python %}
#!/usr/bin/python2
import re
import subprocess
import os

p = subprocess.Popen(["svnversion","/home/user/reactos-work/reactos"], stdout = subprocess.PIPE, stderr = subprocess.PIPE)
p.wait()
m = re.match(r'(|\d+M?S?):?(\d+)(M?)S?', p.stdout.read())
rev = int(m.group(2))
new_name="bootcd-mine-"+str(rev)+".iso"
print new_name
os.rename("/home/user/reactos-work/rosbuild/reactos/bootcd.iso","/home/user/reactos-work/"+new_name)       
{% endhighlight %}






