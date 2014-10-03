---
layout: post
title:  "How to generate sample data on the fly"
date:   2014-10-04 16:57:59
categories: csharp nuget
---
To help me quickly learn new things on the on front-end side, I implemented a small library that generates sample data on the fly.

Suppose you want to learn how angularJS data binding works with ASP.NET MVC WebAPI. You have the following class to play with on the server side:

{% highlight C# %}
class Book
{
    public int Id { get; set; }

    public string Author { get; set; }

    public string Title { get; set; }

    public DateTime ReleaseDate { get; set; }
}
{% endhighlight %}

Now instead of worrying how to create a dozen of fake Books, you can simply write the following:

{% highlight C# %}
var sampleBooks = new DataSampler<Book>()
        .AddPropertyConfiguration(x => x.Id, () => 1)
        .AddPropertyConfiguration(x => x.Author, () => "author")
        .AddPropertyConfiguration(x => x.Title, () => "title")
        .AddPropertyConfiguration(x => x.ReleaseDate, () => DateTime.Now)
        .GenerateListOf(12);
{% endhighlight %}

That will give you a list of twelve sample books. A bit boring, I admit, so how about that:

{% highlight C# %}
int i, j = 0;
var sampleBooks = new DataSampler<Book>()
        .AddPropertyConfiguration(x => x.Id, () => i++)
        .AddPropertyConfiguration(x => x.Author, () => Guid.NewGuid().ToString())
        .AddPropertyConfiguration(x => x.Title, () => Guid.NewGuid().ToString())
        .AddPropertyConfiguration(x => x.ReleaseDate, () => DateTime.Now.AddDays(j++))
        .GenerateListOf(12);
{% endhighlight %}

That's cool but what if you have Author modelled as a sub class?

{% highlight C# %}
class Author
{
    public string FirstName { get; set; }

    public string LastName { get; set; }
}
{% endhighlight %}

No problem, use `AddKnownType()`

{% highlight C# %}
var sampleBooks = new DataSampler<Book>()
        .AddPropertyConfiguration(x => x.Id, () => 1)
        .AddPropertyConfiguration(x => x.Author.FirstName, () => Guid.NewGuid().ToString())
        .AddPropertyConfiguration(x => x.Author.LastName, () => Guid.NewGuid().ToString())
        .WithKnownType(typeof(Author))
        .GenerateListOf(12);
{% endhighlight %}

You can find my library on [github] and on [nuget]. Just paste the following to package manager console to add reference:

`Install-Package DevProductivity_DataTools_DataSampler`

[github]: https://github.com/benetkiewicz/DataSampler/
[nuget]: https://www.nuget.org/packages/DevProductivity_DataTools_DataSampler/
