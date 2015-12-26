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

To make life easier, I created three batch scripts: _backup.bat_, _drop.bat_ and _restore.bat_, that wrap `osql.exe` to do the obvious. But there's a problem. While number of apps I have to support in my day job increases, I need to remember more and more database names because these scripts are dummy and need explicit, correct parameters. I decided that enough is enough and created smarter PowerShell module that includes tab completion for available databases and automatically restores last backup if called without parameters.

{% highlight PowerShell %}
New-SqlBackup -dbName AdventureWorks
Restore-SqlBackup AdventureWorks
{% endhighlight %}

Please make note that this module depends on __SQL Server Management Objects__ API, so you need to have __Client Tools SDK__ for SQL Server (part of SQL Server installation) or __Shared Management Objects__ from the [SQL Server feature pack][ssfp].

### Download

SqlHelpers PowerShell module is available on [github][sqlUtils]

[sqlUtils]: https://github.com/piotratais/SqlHelpersPS/
[ssfp]: https://www.microsoft.com/en-us/download/details.aspx?id=44272
