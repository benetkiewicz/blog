---
layout: post
title: "Roslyn Syntax"
date: "2015-04-14 07:04"
categories: csharp roslyn
---
Creating a reasonable abstraction over C# syntax was one of the key factors to Roslyn success. In compilers world, creating syntax trees is one of the basic concepts and very important step on the way to a binary result file.
Creating syntax tree in Roslyn was even harder task. It not only has to perform all tasks that other compilers perform inside their black boxes but, being _Compiler as a Service_, it must provide clear, easy to understand API and features not directly related to producing binary, like:

* smart, scoped error reporting
* features related to code formatting and refactoring
* helper mechanisms for tree rewriting and traversing

Roslyn aims to fulfill all the above promises inside a `SyntaxTree` class (and its `CSharpSyntaxTree` derivate). That class is a center of gravity which all the syntax supporting code is orbiting. One of the most important things to know about Roslyn syntax trees is that they are immutable. Thanks to immutability SyntaxTree is:

* a lot less likely to contain bugs related to locking and multi-thread access issues
* fast and performant

So let's explore few usage scenarios, for both creating and consuming C# syntax trees.

### Creating syntax trees

#### Create by ParseText()
The most natural way of creating syntax tree from the compiler perspective is to build it from a raw source code. Obviously there's a Roslyn API for this kind of operation:

{% highlight C# %}
string source = "public static class Foo { }";
SyntaxTree syntaxTree = CSharpSyntaxTree.ParseText(source);
{% endhighlight %}

#### Use SyntaxFactory methods
Second, a lot more structured approach, is to build syntax trees with API only, using syntax nodes of different `SyntaxKind` as puzzle pieces that create a big picture. `SyntaxFactory` class contains a bazzilion of methods for each and every C# syntax you can imagine:

{% highlight C# %}
SyntaxTree st = CSharpSyntaxTree.Create(SyntaxFactory.ClassDeclaration("Foo").WithModifiers(SyntaxFactory.TokenList(SyntaxFactory.Token(SyntaxKind.PublicKeyword), SyntaxFactory.Token(SyntaxKind.StaticKeyword))).NormalizeWhitespace());
{% endhighlight %}

#### Use other SyntaxTree parsing methods for parsing individual nodes
As you can see above, creating syntax tree with API for something even a bit more complicated can be a pain. So what if we have an existing syntax tree that comes from a source and it needs a simple modification (let's say a new variable declaration with initializer)? Fortunately that additional syntax node does not have to be created using `SyntaxFactory` building blocks. `SyntaxFactory` can also create a variety of complex syntax node based on "incomplete" code, like this:

{% highlight C# %}
var localVariableDeclaration = SyntaxFactory.ParseStatement("int foo = 123;");
{% endhighlight %}

### Consuming syntax trees

There are at least three methods of exploring syntax trees and we'll explore each of them in details.

#### Evaluating SyntaxKind property

As mentioned before, one of the basic properties of a tree node is its SyntaxKind. This `enum` described every piece of C# code with its keyword or specific language construct and every node supports `IsKind()` method. Consider the following code which prints all properties in a class. Assuming the tree structure and presence of `IsKind()` method, we can write the following code:
{% highlight C# %}
var localVariableDeclaration = SyntaxFactory.ParseStatement("int foo = 123;");
{% endhighlight %}

#### Utilizing OfType<T> method
Another great way of syntax tree traversal is `OfType<T>` method. It is like supercharged `where` clause for filtering tree for a requested node type.
{% highlight C# %}
var localVariableDeclaration = SyntaxFactory.ParseStatement("int foo = 123;");
{% endhighlight %}

#### Implementic your own SyntaxWalker
Last but not least: `CSharpSyntaxWalker`. It is very easy to use, just derive from `CSharpSyntaxWalker` class and override `VisitX()` method, where _X_ is a type of node of your choice. You can override multiple methods, you can override all-taking `Visit()` method, you can aggregate some results inside your walker instance. This is a perfect way of accomplishing some more advanced goals.

{% highlight C# %}
var localVariableDeclaration = SyntaxFactory.ParseStatement("int foo = 123;");
{% endhighlight %}

### Syntax tree modification
Thanks to rich API there's a pletheora or new possibilities when it comes to working with code. The most common usage of syntax tree modification API is creating Analyzers and Code Fixes. Hovewer nothing will stop you from creating your own tools and libraries that will apply code refactorings or even change the code in the fly.

Since syntax trees are immutable, each modification of a tree or a syntax node will produce new instance. Syntax node of various SyntaxKind expose their specific modification entry points that make sense in their scope. So for example `IfStatementSyntax` contains methods `WithStatement()` and `WithCondition()` that will produce new `IfStatementSyntax` object with respective part being altered to a new value. Popular Roslyn code fix example is the one where single statement that follows `if` is being embedded in a block. It nicely illustrates what tree modification is all about.

{% highlight C# %}
// taking ifStatement of type IfStatementSyntax on input
var newIfStatement = ifStatement.WithStatement(SyntaxFactory.Block(ifStatement.Statement));
{% endhighlight %}
