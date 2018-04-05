---
layout: post
title: "Logstash Forwarder Certificate Issues"
date: 2015-05-21 23:22:44 +0300
comments: true
categories: logstash-forwarder ssl certificate logstash
---
This post describes a few certificate errors I experienced using logstash-forwarder.
<!--more-->
When I upgraded logstash-forwarder version I noticed this error:

**Logstash forwarder "Failed to tls handshake with x509: certificate is valid for , not"**.

From a quick search I found out that the new forwarder version requires the certificate to contain a specified CN ([see relevant issue](https://github.com/elastic/logstash-forwarder/issues/221)).

This is how to do it. In this example I used a certificate for domain wildcard.domain.com:

```
openssl req -x509 -subj ‘/CN=*.domain.com/’ -nodes -newkey rsa:2048 -keyout \\
logstash-forwarder.key -out logstash-forwarder.crt
```

However, after creating the certificate the following error message appeared:

**Logstash forwarder “Failed to tls handshake with <logstash IP> x509: certificate has expired or is not yet valid"**.

I validated the certificate using the following command:

```
openssl x509 -in logstash-forwarder.crt -noout -text 
```

And I noticed that the validity duration was set to the default of 30 days.
The way to fix it is to specify a longer validity duration when generating the certificate (for example, 1 year):

```
openssl req -x509 -subj ‘/CN=*.domain.com/’ -batch -nodes -days 365 -newkey rsa:2048 \\
-keyout logstash-forwarder.key -out logstash-forwarder.crt
```

Another error I experienced was the following:

**Logstash forwarder “certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "XX")”**.

This is probably due to mismatching certificates (at least that’s what happened to me), make sure both ends (logstash and logstash-forwarder) use the same certificate and that both services were restarted.


### **Sources** ###
* [https://github.com/elastic/logstash-forwarder](https://github.com/elastic/logstash-forwarder)
* [https://github.com/elastic/logstash-forwarder/issues/221](https://github.com/elastic/logstash-forwarder/issues/221)
* [http://blog.tinle.org/?p=408](http://blog.tinle.org/?p=408)

