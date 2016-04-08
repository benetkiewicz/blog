---
layout: post
title: Advanced late hotfix scenario in Git
date: '2016-04-07 22:56'
categories: git
---

There was a project with the following history.

* 4aa5222 - some other bugfix
* 220a4bd - small feature 2
* ebad147 - important fix 2
* b04860f - small feature 1
* f4a7a5d - some bugfix
* a39ba71 - important fix 1
* e117d73 - something 2
* 5fe39ad - something 1

This project hit production after `5fe39ad - something 1`. Soon we realized that there were two bugs but business decided that they are not so important and we'll wait for fixes until next planned release. Then developers fixed these bugs directly in development branch. Then business changed their minds. How to release hotfix with important fixes 1 and 2 and have a nice development branch history? Here's what I did.

+ Create patches for `a39ba71` and `ebad147`:

```
[development ≡]> git format-patch -1 a39ba71
[development ≡]> git format-patch -1 ebad147
```

+ Revert a39ba71 and ebad147:

```PowerShell
[development ≡]> git revert a39ba71
[development ≡]> git revert ebad147
```

+ Create hotfix branch from release:

```powershell
[development ≡]> git checkout release-1
[release-1 ≡]> git checkout -b hotfix-1
```

+ Apply patches. Powershell is acting out when using less then operator, so some voodoo is required:

```powershell
[hotfix-1 ≡]> cmd.exe /c "git am < C:\temp\0001-important-fix-1.patch"
[hotfix-1 ≡]> cmd.exe /c "git am < C:\temp\0001-important-fix-2.patch"
```

+ Merge to release and development.

I realize that this is more note to self rather then full-blown blog entry but I hope it may help someone in similar situation.
