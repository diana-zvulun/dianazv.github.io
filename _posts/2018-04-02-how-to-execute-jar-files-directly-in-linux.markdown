---
layout: post
title: "How to Execute jar files directly in linux"
date: 2018-04-01T20:00:50+03:00
comments: true
categories: linux java 
---

If you ever wondered how is it that linux can run many programs by simply typing their name in the shell no matter the extension, or like me, you have an operating system that used to run jar files without the preceding "java -jar <jarFile>" and one day it stopped working, then this post is for you.

Linux (I tested it on Centos) has a kernel capability called **binfmt-misc**, which according to wikipedia: 
>
"is a capability of the Linux kernel which allows arbitrary executable file formats to be recognized and passed to certain user space applications".

This means that you can tell your operating system which interpreter should be invoked when running your application by registering it to binfmt-misc.

### **So how do I register jar files?** ###
Actually, if you have JDK package installed, you should already have a file that does that for you:
```
/etc/init.d/jexec
```

Once you start this init.d file it registers the jexec interpreter with the binfmt-misc system and makes it possible to run jar files like any other known executable.
```
/etc/init.d/jexec start
```
When installing the JDK package it also adds this service using chkconfig so that it will start on every boot.



### **Sources** ###

* [https://en.wikipedia.org/wiki/Binfmt_misc](https://en.wikipedia.org/wiki/Binfmt_misc)
* [https://www.kernel.org/doc/Documentation/admin-guide/binfmt-misc.rst](https://www.kernel.org/doc/Documentation/admin-guide/binfmt-misc.rst)

