---
layout: post
title: "Puppet 3 upgrade with puppet-lint and Catalog Preview"
date: 2020-02-15 00:00:00 +0300
comments: true
categories: puppet puppet-lint catalog-preview
---
Puppet upgrade from version 3 to version 4 and higher is not a trivial task. It requires some preparations and making sure your agents will not break after the migration.
To do that you need to first migrate your modules from the old version to the new one. In this post I will show you 2 tools I used in my migration that made my life easier.
<!--more-->

### **Puppet-lint:** ###
Before running catalog preview tool to see what will break in my puppet 3 modules (more on that later in this post), I ran [puppet-lint](http://puppet-lint.com/) which is a nice tool that checks your modules/manifests for syntax according to the [puppet style guide](https://puppet.com/docs/puppet/latest/style_guide.html):

```puppet-lint /etc/puppetlabs/code/environments/production/modules/mymodule/ --with-context --no-140chars-check```

This tool can also try to fix your errors if you add the --fix option to the command:
```
[~]#puppet-lint /etc/puppetlabs/code/environments/production/modules/mymodule/ --with-context --no-140chars-check --fix
FIXED: unquoted resource title on line 34
FIXED: unquoted resource title on line 46
FIXED: double quoted string containing no variables on line 58
FIXED: double quoted string containing no variables on line 58
FIXED: indentation of => is not properly aligned (expected in column 13, but found it in column 12) on line 113
FIXED: indentation of => is not properly aligned (expected in column 13, but found it in column 12) on line 114
FIXED: trailing whitespace found on line 107
```
After my modules were updated according to the style guide I continued to the next step which is to run the catalog preview.

### **Catalog preview:** ###
This is a [tool](https://github.com/puppetlabs/puppetlabs-catalog_preview) written by puppetlabs to make the migration from puppet3 parser to puppet4 future parser easier. It can compile catalogs for the same node in each environment to compare them both and output a report that shows if there are any changes. This is a game changer, since it was always very difficult to manage puppet upgrades because most of the time you don’t know what exactly stopped working and why, which could be dangerous in a production environment.
* First I Installed a new puppet master version 3.8.1 that is not serving requests and installed the catalog-preview module on it : ```puppet module install puppetlabs-catalog_preview```
* Before executing the puppet preview command for a node, you need to make sure this node communicated **at least once** with the puppet master and has its facts.yaml under: ```/var/lib/puppet/yaml/facts/```
* Decide on which environment you want to execute the preview command. I created a new baseline environment that was a copy of my production environment since I didn’t want to make changes in production. If you create only one environment you can use the ```--migrate 3.8/4.0``` option and the preview command compares this one environment to itself but with different parsers. The baseline env uses puppet 3 parser (current) and the preview env uses the future parser.
* Run the preview command on a selected node: ```puppet preview --baseline-environment baseline_env <servernamefqdn> --migrate 3.8/4.0 --view overview``` and make fixes as you go along.


The output has all kinds of error messages which are detailed in the [documentation](https://github.com/puppetlabs/puppetlabs-catalog_preview#understanding-output-and-results), but the desired state is both catalogs having the same resources declared, if you see your new environment is not aligned with your previous one, you should check why and make sure they are the same.

Good Luck!

### **Sources** ###
* [https://puppet.com/docs/puppet/4.10/upgrade_major_pre.html](https://puppet.com/docs/puppet/4.10/upgrade_major_pre.html)
* [https://engineering.skroutz.gr/blog/upgrading-puppet3-to-puppet4/](https://engineering.skroutz.gr/blog/upgrading-puppet3-to-puppet4/)
* [https://www.example42.com/2017/01/23/existing-code-on-puppet4/](https://www.example42.com/2017/01/23/existing-code-on-puppet4/)
