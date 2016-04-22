---
layout: post
title: "Jenkins nodes and labels solve VPN and Selenium issues"
date: "2016-04-16 23:01"
categories: jenkins ci selenium
---
While setting up new windows server application stack in my company I came across two serious issues:
1. My company works as contractor for Nice Invincible Corporation, which means that source code repository is often accessible only to developers because of VPN and security policy that VPN can only run in "human" context - no service accounts are permitted.
2. Selenium tests need to be run from entity that can do _desktop interaction_. By default, Jenkins on windows runs as windows service, so Selenium tests miserably fail.

#### Source code repository behind VPN solution
If you use git and git only, the obvious option is to host second remote repo inside of your company and push code to both remotes in parallel. Things get complicated if you also host code in SVN, TFS and CVS (sic!).
Jenkins nodes to the rescue! Long story short: setup nodes on developer machines and configure jenkins to run code fetch tasks only on these machines using labels. Here is step by step instruction:
1. Setup A
2. Setup B
3. Setup C.

#### Run selenium tests from Jenkins
As mentioned before, Selenium tests need desktop interaction and windows services do not interact with desktop nicely, even though there is _allow service to interact with desktop_ option in services settings window. Common solution that you can find in the internet is to run jenkins from war, instead of host it as a windows service. This is nice if you use it locally but for a server solution windows service is much better because it will start automatically after each server restart.

Nodes to the rescue again! Create new node and run it from the console as _Java Web Start_. Apply Selenium label. Configure jenkins to run Selenium tests on nodes with _Selenium_ labels. You can even run this node on the same machine where Jenkins service is hosted. Agreed, after server restart, Selenium tests will not be run but all other Jenkins jobs will work as expected.
