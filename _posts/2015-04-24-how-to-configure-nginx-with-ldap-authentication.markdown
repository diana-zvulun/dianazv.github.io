---
layout: post
title: "How to configure Nginx with LDAP authentication"
date: 2015-04-24 11:19:34 +0300
comments: true
categories: nginx ldap
---
Here I will show how you can setup Nginx with user authentication through Active Directory.
<!--more-->

### **Build Nginx from source with ldap module (tested on CentOS 6)** ###


Download Nginx source from [nginx.org/download](http://nginx.org/download/):

```
    wget http://nginx.org/download/nginx-1.7.9.tar.gz 
```

Download nginx-auth-ldap third party module from https://github.com/kvspb/nginx-auth-ldap: 

You can do it in 2 ways:

```
    git clone https://github.com/kvspb/nginx-auth-ldap.git
    wget https://github.com/kvspb/nginx-auth-ldap/archive/master.zip
```

Install tools that are required for the build process:

```
    yum groupinstall "Development Tools"
    yum install openssl-devel
    yum install pcre-devel
```

Untar/unzip the files you’ve downloaded and enter the Nginx folder:

```
    cd nginx-1.7.9
```

Now we will build Nginx from source. Mind that you need to change the path for the nginx-auth-ldap module in the "--add-module=" section:

```
    ./configure --user=nginx                              \\
                --group=nginx                             \\
                --prefix=/etc/nginx                       \\
                --sbin-path=/usr/sbin/nginx               \\
                --conf-path=/etc/nginx/nginx.conf         \\
                --pid-path=/var/run/nginx.pid             \\
                --lock-path=/var/run/nginx.lock           \\
                --error-log-path=/var/log/nginx/error.log \\
                --http-log-path=/var/log/nginx/access.log \\ 
                --with-http_gzip_static_module            \\
                --with-http_stub_status_module            \\
                --with-http_ssl_module                    \\
                --with-pcre                               \\
                --with-file-aio                           \\
                --with-http_realip_module                 \\
                --add-module=/nginx-auth-ldap-master/     \\
                --with-ipv6                               \\
                --with-debug                              \\
    
    
    make
    make install
```


Nginx init file:

Nginx wiki provides an init file for RedHat: http://wiki.nginx.org/RedHatNginxInitScript

Copy the file and place it under /etc/init.d/nginx

Make the file executable: 

```
chmod +x /etc/init.d/nginx
```

And add it to startup at boot:

```
chkconfig --add nginx
chkconfig nginx on
```

You can now check that Nginx is available on the default port by going to http://FQDN, this is the output you should see:

![nginx_image](/images/nginx.png)

### **Package Management** ###
After you have everything working, you can package your work and install it easily on other servers. 

I use this great tool: https://github.com/jordansissel/fpm which makes packaging that much easier.

### **Adding LDAP module to Nginx configuration file** ###

Open nginx.conf file for editing:

```
   vi /etc/nginx/nginx.conf
```

Add the following:

```  
 
 ##LDAP module configuration under http section:
   http {
    
		auth_ldap_cache_enabled on;
		auth_ldap_cache_expiration_time 10000;
		auth_ldap_cache_size 1000;
 
 ##Configuration of your LDAP server, I configured a specific group to have access:
		ldap_server LDAP1 {
			url "ldaps://<ldap_server_name>:<ldap_server_port>/dc=xxx,dc=xxx?sAMAccountName?sub?";
			binddn "ldap_user_name";
			binddn_passwd "ldap_user_password";
			connect_timeout 5s;
			bind_timeout 5s;
			request_timeout 5s;
			satisfy any;
			group_attribute member;
			group_attribute_is_dn on;
			require group "CN=LDAP_Group_Name,OU=xxx,DC=xxx";
		}

 ##You can chain another ldap_server directive here for redundancy
 
		server {
		  listen                *:443 ssl ;

 ##Login message that the user will see when entering your website:
		  auth_ldap "Please enter your ldap user";
		  auth_ldap_servers LDAP1;
 ##If you have configured another ldap server, you can chain it here  


		}
	 }

    
```

Note that you can place the **auth_ldap** directive under **server** directive, or under a specific **location** directive in case you have multiple locations and you only want one of them to require authorization.


### **Sources** ###

* [https://calvin.me/nginx-ldap-http-authentication/](https://calvin.me/nginx-ldap-http-authentication/)
* [http://www.allgoodbits.org/articles/view/29](http://www.allgoodbits.org/articles/view/29)
* [https://github.com/kvspb/nginx-auth-ldap](https://github.com/kvspb/nginx-auth-ldap)