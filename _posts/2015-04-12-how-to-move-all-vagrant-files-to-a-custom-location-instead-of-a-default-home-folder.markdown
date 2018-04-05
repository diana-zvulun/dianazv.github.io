---
layout: post
title: "Move all vagrant files to a custom location instead of a default home folder"
date: 2015-04-12 22:42:21 +0300
comments: true
categories: 
---
When you don't want Vagrant to fill your C: drive with boxes & logs, here is how to do it :
<!--more-->

### Change it on Oracle VM VirtualBox Manager ###
Open "Oracle VM VirtualBox Manager">  File> Settings> Default machine folder > [New Location]


### Change the environment variables ###

	ENV_VARIABLES: 
	VAGRANT_HOME=<New Location>
	VBOX_LOG_DEST=<New Location>
	VBOX_USER_HOME=<New Location>