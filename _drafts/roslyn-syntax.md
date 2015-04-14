---
layout: post
title: "Roslyn Syntax"
date: "2015-04-14 07:04"
---
Creating a reasonable abstraction over C# syntax was one of the key factors to Roslyn success. In compilers world, creating syntax trees is one of the key concepts and very important step on the way to a binary result file.
Creating syntax tree in Roslyn was even harder task. It not only has to perform all tasks that other compilers perform inside their black boxes but, being Compiler as a Service, it must provide clear, easy to understand API and features not directly related to producing binary, like:

* smart, scoped error reporting
* features related to code formatting and refactoring
* helper mechanisms for tree rewriting and traversing

Roslyn aims to fulfill all the above promises inside a `SyntaxTree` class (and its CSharpSyntaxTree derivate). That class is a center of gravity which all the syntax supporting code is orbiting. One of the most important things to know about Roslyn syntax trees is that they are immutable. Thanks to immutability SyntaxTree is:

* a lot less likely to contain bugs related to locking and multi-thread access issues
* fast and performant

So let's explore few usage scenarios, for both creating and consuming C# syntax trees.

## Creating syntax trees
The most natural way of creating syntax tree from the compiler perspective is to build it from a raw source code. Obviously there's a Roslyn API for this kind of operation:

{% highlight C# %}
string source = "class Foo { public int Bar { get; set; } }";
SyntaxTree syntaxTree = SyntaxTree.ParseText(source);
{% endhighlight %}

Second, a lot more structured approach, is to build syntax trees with API only, using syntax nodes of different `SyntaxKind` as puzzle pieces that create a big picture. `SyntaxFactory` class contains a bazzilion of methods for each and every C# syntax you can imagine:

{% highlight C# %}
var syntaxTree = SyntaxFactory.CreateClass("Foo");
{% endhighlight %}

As you can see above, creating syntax tree with API for something even a bit more complicated can be a pain. So what if we have an existing syntax tree that comes from a source and it needs a simple modification (let's say a new variable declaration with initializer)? Fortunately that additional syntax node does not have to be created using `SyntaxFactory` building blocks. `SyntaxFactory` can also create a variety of complex syntax node based on "incomplete" code, like this:

{% highlight C# %}
var localVariableDeclaration = SyntaxFactory.ParseVarDecl("int foo = 123;");
{% endhighlight %}

## Consuming syntax trees

* OfType<>
* DescendantNodes().SyntaxKind
* Walker
