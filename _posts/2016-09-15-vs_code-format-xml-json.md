---
layout: post
title: Use VS Code to prettify XML and JSon code
date: '2016-09-15 23:00'
categories: tips code json
---

While debugging, you often need to analyze a random piece of XML or JSon. Most of the time it is not pretty-printed, it is a single line, ugly blob which makes it hard to analyze. On my second [blog][a2e81bda] I presented a tool which I use for XML formatting but since VS Code makes its way to most .NET developers machines, I thought it would be nice to have these features in Code as well. Additionally, JSon support is present in Code by default, so it makes it one to rule them all.

So how to quickly format a random piece of XML or JSon in Code?

![VS Code format](images/vs_code_language_mode.png)

Create a new tab (`ctrl + n`), paste your code snippet (`ctrl + v`). Notice that until you save it, it will not be recognized as XML or JSon, so _Format Code_ option, which we'll cover in a minute, will not be available. You probably don't want to save this file (file extension would hint code what is the type of data), so you need to use _Change Language Mode_ (`ctrl + k, m`) option from the command palette (`ctrl + shift + p`) and choose JSon or XML. By the way, XML is not supported by default, so make sure you have [XML Tools][b874bbc8] extension installed. Now, since VS Code is aware of context, hit (`ctrl + shift + p`) again and choose _Format Code_.

[a2e81bda]: https://devproductivity.wordpress.com/2012/03/29/xml-formatting-and-indenting-made-easy/ "blog"

[b874bbc8]: https://marketplace.visualstudio.com/items?itemName=DotJoshJohnson.xml "Xml Tools"
