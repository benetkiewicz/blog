---
layout: post
title: "Announcing release of Roslinq - library for C# code queries"
date: "2015-06-10 11:03"
---

## Roslinq
Roslinq is a Roslyn based component that enables queries over C# source code.

### Features
Roslinq currently supports:

* MSBuild based workspaces (_.sln_, _.csproj_)
* class queries
* method queries

### Examples

Private static methods taking `int` parameter input:

{% highlight C# %}
var codeQuery = new ProjectQuery(@"path\to\RoslinqTestTarget.csproj");
var result = codeQuery
    .Classes
    .Methods
    .WithModifier(MethodModifier.Static)
    .WithModifier(MethodModifier.Private)
    .WithParameterType<int>()
    .Execute();
{% endhighlight %}

HTTP POST handlers in MVC Controllers:

{% highlight C# %}
var codeQuery = new ProjectQuery(@"path\to\RoslinqTestTarget.csproj");
var postControllerActions = codeQuery
    .Classes.InheritingFrom<Controller>()
    .Methods.WithAttribute<HttpPostAttribute>()
    .Execute();
{% endhighlight %}

### Nuget package
```
Install-Package Roslinq -pre
```
