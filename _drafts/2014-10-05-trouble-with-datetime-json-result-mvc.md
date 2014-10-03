---
layout: post
title:  "JSon result and datetime"
date:   2014-10-05 16:57:59
categories: csharp javascript json
---
In ASP.NET MVC you can return a number of different results from your controller, one of them is JSonResult. It automatically performs object serialization to JSon format and pushes serialized object to requestor. It works pretty good overall but there's a problem with DateTime values.

People are doing all sort of things to fight this off, see stack overflow discussion [here]
* regex parsing
* custom results, deriving from JSonResult
* custom libraries, for example taking advantage of $.parseJSON from JQuery

It will all work, just pick your poison. My choice recently was to return string version of the date and parse it back on the client side.

[here]: http://stackoverflow.com/questions/726334/asp-net-mvc-jsonresult-date-format
