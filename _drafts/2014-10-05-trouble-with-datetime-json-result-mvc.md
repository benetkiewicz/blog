---
layout: post
title:  "JSon result and datetime"
date:   2014-10-05 16:57:59
categories: csharp javascript json
---
In ASP.NET MVC you can return a number of different results from your controller, one of them is JSonResult. It automatically performs object serialization to JSon format and pushes serialized object to requestor. It works pretty good overall but there's a problem with DateTime values.
Consider the following controller class:

{% highlight C# %}
public class DateController : Controller
{
    [HttpGet]
    public ActionResult Index()
    {
        return Json(new TimeContainer(DateTime.Now), JsonRequestBehavior.AllowGet);
    }
}
{% endhighlight %}

and the model is:

{% highlight C# %}
public class TimeContainer
{
    public TimeContainer(DateTime seed)
    {
        this.OriginalValue = seed;
        this.StringValue = seed.ToString("s");
    }

    public string StringValue { get; set; }

    public DateTime OriginalValue { get; set; }
}
{% endhighlight %}

Result of the controller `Index` action looks like that:
{% highlight javascript %}
{
    "StringValue": "2014-10-04T14:04:12",
    "OriginalValue": "/Date(1412424252981)/"
}
{% endhighlight %}

Note that `OriginalValue` format is not the first thing you would expect to be returned, right?

People are doing all sort of things to fight this off, see stack overflow discussion [here]

* regex parsing
* implementing custom result classes, deriving from JsonResult
* custom libraries, for example taking advantage of `$.parseJSON` from JQuery

It will all work, just pick your poison. My recent choice was to return string version of the date and parse it back on the client side.

What's also interesting, ASP.NET WebAPI controller

{% highlight C# %}
public class DateTimeController : ApiController
{
    public TimeContainer Get()
    {
        return new TimeContainer(DateTime.Now);
    }
}
{% endhighlight %}

will return that:

{% highlight javascript %}
{
    "StringValue": "2014-10-04T14:18:40",
    "OriginalValue": "2014-10-04T14:18:40.2338511+02:00"
}
{% endhighlight %}

[here]: http://stackoverflow.com/questions/726334/asp-net-mvc-jsonresult-date-format
