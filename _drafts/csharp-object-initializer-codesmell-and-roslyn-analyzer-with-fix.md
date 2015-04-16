---
layout: post
title: "CSharp object initializer codesmell and Roslyn analyzer with fix"
date: "2015-04-13 23:26"
---
One of the most annoying and non-obvious C# code smells is object initializer. I really hate them.

* They look ugly in the code, it is hard to format them in a proper way
* They are harder to debug (especially in VS 2010), when you put a breakpoint on a single initializer line, Visual Studio puts breakpoint on entire initializer
* They blur stack trace. If you have complex object initializer with nested properties being read (or God forbids - methods), your stack trace is pointed to a first line of initialization statement.

These are the three main reasons which make me remove object initializers from code every time I can. Unfortunately this is a pain. R# seems to be fine with object initializer and refactoring by hand is clunky and not a lot of fun. Fortunately there's a new guy in town - Roslyn.

Recently I got my hands dirty and created a simple Roslyn analyzer and Code Fix for object initializers. From now on I can refactor my code to use cosher properties access with just a few key strokes.

My solution will detect and fix only the simplest form of the object initializer construct used for local variables. It explicitly ignores:

* nested object initializers: `Foo f = new Foo() { F = new Foo() { F = null } }`
* multiple variable statements: `Foo g = null, f = new Foo() { F = null }`;
* object initializers used in parameters, property assignments: `ProcessFoo(new Foo { F = null });`

Code Fix is a lot more complex then most examples that you can find in Roslyn tutorials and let me tell you why. Most code fixes operate in a single context and they don't need to go beyond a language construct being analyzed and fixed. For example, the [popular if braces analyzer][36f76dca] fixes everything inside a particular IfStatementSytax and does not have to go beyond that syntax node. It just takes expression and embeds it in a block if necessary. For me, dealing with ObjectInitializerSyntax is not enough. I need to remove it but I also need to add new statements right after variable initialization that correspond to property initialization that just have been removed. That means that I need to play with a code block containing given local variable initialization statement.

Applying few modifications on a immutable syntax tree has an interesting implication. After you apply your first modification to, let's say code block, all your references to nodes in the original syntax tree are lost (or I should rather say: inapplicable). Consider the following example:
![roslyn block modification](\images\roslyn-block-modification.png)

After you perform the insert operation on Block, you cannot simply perform replace Statement1 with Statement4 in Block':

<code>Block'.Replace(Statement1, Statement4)</code>

There is no Statement1 in Block' anymore! You must recalculate reference to Statement1' inside Block' somehow.

Fortunately Roslyn covers the above scenario in a very convenient way.

[36f76dca]: http://jeremybytes.blogspot.com/2014/12/updates-to-ifbracesanalyzer-code-fix.html
