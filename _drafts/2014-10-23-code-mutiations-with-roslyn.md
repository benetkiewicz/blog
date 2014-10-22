---
layout: post
title:  "Code mutation with Roslyn"
date:   2014-10-22 22:57:59
categories: csharp roslyn
---

On the [devday] conference I attended in September, there was this guy, [Seb Rose], who asked an interesting question:

`Is coverage a good metric?`

When you think about it, it may not be the best. Assuming you have the following code to test:

{% highlight C# %}
public class Calculator
{
    public static int MultiplyByTwo(int number)
    {
        const int two = 2;
        return number*two;
    }
}
{% endhighlight %}

The following unit test will give you a pretty good coverage:

{% highlight C# %}
Debug.Assert(Calculator.MultiplyByTwo(0) != null);
{% endhighlight %}

But is that good? Obviously no. What if you could create code mutation and run the test again? Code mutation would be almost the same as before but with a subtle change, which would slightly change how the algorithm works and should be detected by unit test.
With that magic engine in place, we could define our new metric as percentage of failing tests, which are being run against all kinds of code mutations. Ideally, the best result is when only the original code works and the simplest semantical change breaks the test.

This idea became my __idée fixe__ for the last month and it encouraged my to make a reasearch on a scary topic, which is .NET compiler platform (__[Roslyn]__). Roslyn is advertised as Compiler as a Service which in my mind translates to API that allows you to work with a source code on both syntax and semantic level.

[devday]: http://devday.pl
[Seb Rose]: https://twitter.com/sebrose
[Roslyn]: http://msdn.microsoft.com/en-us/vstudio/roslyn.aspx
