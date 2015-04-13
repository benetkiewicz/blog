---
layout: post
title: "CSharp object initializer codesmell and Roslyn analyzer with fix"
date: "2015-04-13 23:26"
---
One of the most annoying and non-obvious C# code smells is object initializer. I really hate them.

* They look ugly in the code, it is hard to format them in a proper way
* They are harder to debug (especially in VS 2010), when you put a breakpoint on a single initializer line, Visual Studio puts breakpoint on entire initializer
* They blur stack trace. If you have complex object initializer with nested properties being read (or God forbids - methods), your stack trace is pointed to a first line of initialization statement

These are the three main reasons which make me remove object initializers from code every time I can. Unfortunately this is a pain. R# seems to be fine with object initializer and refactoring by hand is clunky and not a lot of fun. Fortunately there's a new guy in town - Roslyn.

Recently I got my hands dirty and created a simple Roslyn analyzer and Code Fix for object initializers. From now on I can refactor my code to use cosher properties access with just a few key strokes.

Analyzer is really simple. It just search for Roslyn build-in object initialize syntax. The only rocket science present here is that I don't want to deal with nested initializers, like that:

Code Fix is a lot more complex then most examples that you can find in Roslyn tutorials and let me tell you why. Most code fixes operate in a single context and they don't need to go beyond a language construct being analyzed and fixed. For example the immortal single if statement example fixes everything inside a particular ifstatementsytax and does not have to go beyond that syntax node. It just takes expression and embeds it in a block if necessary. For me, dealing with objectinitializersyntax is not enough. I need to remove it but I also need to add new statements right after variable initialization that correspond to property initialization that just have been removed. Thant means that I need to play with a code block containing goven object initialization.
