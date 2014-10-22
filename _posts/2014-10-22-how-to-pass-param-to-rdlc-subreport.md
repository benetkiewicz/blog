---
layout: post
title:  "How to pass a parameter to a RDLC subreport"
date:   2014-10-22 22:57:59
categories: csharp rdlc
---
This is one of these posts to remember. Every now and then I am forced to design a bit more complex report that has to pass parameters to its subreports. There are few steps to follow and I don't want to discover this again in few months, so here's how it goes.

Edit subreport first. In _Report data_ window right click on _Parameters_ node, click _Add Parameter_ option and fill in the configuration options as on the image below.

![RDLC subreport new param]({{ site.url }}images/subreport_new_param.png)

On the main report, right click subreport and select _Subreport Properties_. Go to _Parameters_ tab, click _Add_ and set the same name of the parameter as in the previous step. In the value column, set what's appropriate. Most likely it will be a column from your data set or some expression.

![RDLC report new param]({{ site.url }}images/report_new_param.png)

If in both report and subreport name and type will match, data should be passed into the subreport without any issues.
