---
layout: post
title: "How to Query Active Directory Using Ldapsearch"
date: 2018-03-31T18:20:50+03:00
comments: true
categories: ldap ldapsearch activedirectory
---
First you need to install the package that provides ldapsearch command:

```
yum install openldap-clients
```

Then run the following command according to your environment:
<!--more-->

```
ldapsearch -x -h <active directory server> -D "<user@domain.com>" -W -b \\ 
"CN=<Common Name>,OU=<Organizational Unit>,DC=<Domain Component>,DC=<Domain Component>"
```

* **Common Name:** can be user/group

* **Organizational Unit:** the trees the common name is under. You can also concatenate several OU’s if needed. For example- ```OU=<Organizational Unit 1>,OU=<Organizational Unit 2>```

* **Domain Component:** the domain (DC=domain,DC=com)
