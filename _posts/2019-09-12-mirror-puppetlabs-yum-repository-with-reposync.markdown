---
layout: post
title: "Mirror puppetlabs yum repository with reposync"
date: 2019-09-12 00:00:00 +0300
comments: true
categories: reposync yum puppet
---
Some systems can't be connected to the Internel due to security reasons. In that case you will need to use a local yum repository in your environment, and every once in a while you will need to update this repository. In this post I will show you how to mirror puppetlabs yum repository to a local one using reposync command.
<!--more-->

### **Install the repository package from puppetlabs** ###
```
yum install wget
wget https://yum.puppet.com/puppet6-release-el-7.noarch.rpm
yum loaclinstall puppet6-release-el-7.noarch.rpm
```

This will place a `.repo` conf under - `/etc/yum.repos.d` and a gpg key under `/etc/pki`:
```
[~]# cat /etc/yum.repos.d/puppet6.repo
[puppet6]
name=Puppet 6 Repository el 7 - $basearch
baseurl=http://yum.puppetlabs.com/puppet6/el/7/$basearch
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-puppet6-release
enabled=1
gpgcheck=1
```

### **Add puppetlalbs keys in order to verify the packages** ###

I followed [these](https://puppet.com/docs/puppet/latest/puppet_platform.html#task-6116) instructions. Make sure you add the relevant key for you.
```
gpg --keyserver pgp.mit.edu --recv-key 7F438280EF8D349F
gpg --list-key --fingerprint 7F438280EF8D349F
```
The fingerprint of the Puppet release signing key is 6F6B 1550 9CF8 E59E 6E46 9F32 7F43 8280 EF8D 349F. Ensure the fingerprint listed matches this value.
reposync also needs this key in the rpm keystore according to [this](https://serverfault.com/questions/919390/how-to-reposync-saltstack-reposync-failing-with-error-message-removing) issue. 
```
rpm --import  ///etc/pki/rpm-gpg/RPM-GPG-KEY-puppet6-release
```
Verify the key was imported successfully:
```
rpm -q  gpg-pubkey --qf '%{NAME}-%{VERSION}-%{RELEASE}\t%{SUMMARY}\n'
```

### **Install reposync and Sync the repository to your local machine** ###
```
yum install yum-utils
reposync --gpgcheck --repoid=puppet6
createrepo . --workers=4
``` 


### **Sources** ###

* [https://puppet.com/docs/puppet/latest/puppet_platform.html](https://puppet.com/docs/puppet/latest/puppet_platform.html)
* [https://serverfault.com/questions/919390/how-to-reposync-saltstack-reposync-failing-with-error-message-removing](https://serverfault.com/questions/919390/how-to-reposync-saltstack-reposync-failing-with-error-message-removing)
* [https://bencane.com/2013/04/15/creating-a-local-yum-repository/](https://bencane.com/2013/04/15/creating-a-local-yum-repository/)
* [https://access.redhat.com/solutions/23016](https://access.redhat.com/solutions/23016)
