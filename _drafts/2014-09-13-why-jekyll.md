---
layout: post
title:  "Why Jekyll?"
date:   2014-09-13 17:00:59
---

Why I started a blog based on [__Jekyll__](http://jekyllrb.com/) and hosted on [__Github Pages__](https://pages.github.com/)?

* I like the idea - blog engine based on file system content, automatically building HTML from markdown files
* Github pages are really cool, I love using git and now I can deploy new content with a push!
* Opportunity to learn a bit about Ruby and Python
* Wordpress seems too regular-user-ish and I don't have time to write blog engine from scratch. With Jekyll I have programming without programming ;)
* I want to master markdown syntax

Setup wasn't super easy though. I made my life a bit harder on purpose because I wanted to have jekyll working locally. This way I can catch issues quickly. I followed [this guide](https://help.github.com/articles/using-jekyll-with-pages). I ran into few problems during installation, which were probably common rookie mistake for hard-core .net programmer and windows user as myself.

First, no matter what I did, I couldn't install json gem, I received the following error:

`ERROR: Failed to build gem native extension.`

The fix was to reinstall ruby to a folder without spaces. Initially I installed into `c:\program files (x86)\`. Silly me!

Second error happened while running jekyll for the fist time. It was

``Liquid Exception: undefined method `[]' for nil:NilClass in _posts``

This was resolved by installing python 2.7.8 instead of initially installed 3.4.1.

So few things to remember while setting up jekyll locally:
* install ruby and python into directory which does not contain spaces
* bleeding edge software is not always the best choice. Use python 2.X
* there may be issues with console emulators like __console2__ or __conemu__. Use good old windows command prompt if you'll see any unexplainable errors.
