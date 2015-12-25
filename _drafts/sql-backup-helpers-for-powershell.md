---
layout: post
title: "SQL Backup Helpers for PowerShell"
date: "2015-12-25 23:44"
---

My workflow with SQL often looks like that:

* backup
* hack, hack, hack
* break something important
* restore
* repeat

To make life easier, I created three batch scripts: _backup.bat_, _drop.bat_ and _restore.bat_, that wrap `osql.exe` to do the obvious. But there's a problem. While number of apps I have to support in my day job increases, I need to remember more and more database names because these scripts are dummy and need explicit, correct parameters. I decided that enough is enough and created smarter PowerShell module that includes tab completion and automatically restores last backup if called without parameters. I hope to become employee of the month again with this help ;)

Go ahead. [try it][sqlUtils]

[sqlUtils]: https://github.com/piotratais/SqlHelpersPS/
