---
layout: post
title: 'Fun with F# unit testing'
date: '2015-07-02 12:14'
categories: fsharp testing nunit
---

It is amazing how hard functional programming is for an average csharp guy, and yet how easy it is to express logic in this language.
{% highlight fsharp %}
open NUnit.Framework
let equal arg1 arg2 =
    arg1 = arg2
let should f x y =
    let c = f x
    Assert.IsTrue(c y)
[<Test>]
let ``two plus two should equal four`` =
    1 + 1 |> should equal 2
{% endhighlight %}
The above code is inspired by the awesome [FsUnit][fsunit] project.

[fsunit]: https://github.com/fsprojects/FsUnit
