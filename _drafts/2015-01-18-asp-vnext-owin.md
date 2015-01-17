---
layout: post
title:  "Why ASP vNext configuration is what it is"
date:   2015-01-18 17:00:59
---

This is the bare-bones ASP.NET MVC 6 (vNext) app:

{% highlight C# %}
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseMvc();
    }
}

[Route("api/[controller]")]
public class HelloWorldController : Controller
{
    [HttpGet]
    public string Index()
    {
        return "hello world!";
    }
}
{% endhighlight %}

When I first saw it I thought: cool. But then I thought: hey, what the hell are all these `app.Use()` statements. Why do they look like that, how does that work and how will I know what should I use()? For quite a bit of time that code snippet from _Startup_ class remained kind of a black box for me, I just assumed that this is how it suppose to be and asked no questions.

Until recently when I learned about OWIN and things started to make sense.
