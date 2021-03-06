---
layout: post
title:  "Creating a Chroot"
date:   2016-09-24 21:09:32 -0300
categories: GNU/Linux tricks
---

In this site, I'm using [Jekyll](https://jekyllrb.com/). I wanted to test 
Jekyll on a host running Ubuntu 14.04 LTS, but the version of Jekyll in 
that distribution is rather old (0.11.2-1 when current version is 3.2.1).
However, I didn't want to upgrade the hole system just because of that, so I
remember an old useful trick, that it is very useful in situations like this: 
creating a chroot. I learnt about it while
[installing Gentoo](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base)
(a source-based GNU/Linux distribution).

A [chroot](https://en.wikipedia.org/wiki/Chroot) on Unix operating systems is an operation that changes the apparent 
root directory for the current running process and its children. 
A program that is run in such a modified environment cannot name (and therefore 
normally cannot access) files outside the designated directory tree. 

You can install a completely different flavor of GNU/Linux inside the chroot.
[Here](https://wiki.debian.org/chroot) you can find some useful
documentation on how to do that. In particular, you can use
[debootstrap](https://www.debian.org/releases/stable/amd64/apds03.html.en)
for installing a Debian system (or its derivatives like Ubuntu) inside
another. For instance, I have used debootstrap for installing  Ubuntu 
Xenial Xerus (16.04) instance (which provides the updated jekyll package
that I needed) inside my Ubuntu 14.04 system, by running

{% highlight bash %}
debootstrap xenial /xenial-root
{% endhighlight %}

You need to mount several pseudo filesystems before entering into the 
the chrooted environment.using the command chroot. I am using the following
script for doing that.

{% highlight bash %}
#!/bin/sh -v
# Script to enter into the xenial chroot
mount proc xenial-root/proc -t proc
mount sysfs xenial-root/sys -t sysfs
mount --bind /dev/pts /xenial-root/dev/pts
cp /etc/hosts xenial-root/etc/hosts
chroot xenial-root /bin/bash
{% endhighlight %}

**Tip**: if you install a Debian-derived system (like Ubunutu) is convenient to
create a file **/etc/debian_chroot** file with a name for your chroot
environment. This will be shown in the shell prompt so you can easily know
whether you are inside the chroot or not, thanks to the following
magic in **/etc/bash.bashrc** which sets the PS1 environment variable.

{% highlight bash %}
# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, overwrite the one in /etc/profile)
PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
{% endhighlight %}

Note that a chroot has several security limitations (see the [Wikipedia page
for chroot](https://en.wikipedia.org/wiki/Chroot) ). If you need something
more secure, there are other options like using
[Docker](https://www.docker.com/) or using a virtual machine (with
[Virtualbox](https://www.virtualbox.org/) or
[Quemu](http://wiki.qemu.org/Main_Page).

Note also that you will be still running the same kernel of the host system.
This places some limitations (The chroot technique won't work if the system 
that you want to run inside it needs some newer features in the kernel that
are not available in the host!).

**Tip**: By the way, if you just want to install ruby and jekyll, after some trials, I have found 
[this tutorial](https://gorails.com/setup/ubuntu/14.04) where it is clearly 
explained how to install the current version of ruby from the sources for your user, 
without super-user (root) privileges. After that,

{% highlight bash %}
gem install jekyll bundler
{% endhighlight %}

as explained in the [Jeckyll web page](https://jekyllrb.com/). I recommend 
this way of installation.


 
