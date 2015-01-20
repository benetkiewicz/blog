---
layout: post
title:  "Explaining ASP.NET vNext Configure method on OWIN example"
date:   2015-01-18 17:00:59
categories: csharp vNext owin
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

<blockquote>
OWIN defines a standard interface between .NET web servers and web applications. The goal of the OWIN interface is to decouple server and application, encourage the development of simple modules for .NET web development, and, by being an open standard, stimulate the open source ecosystem of .NET web development tools.
</blockquote>

OWIN philosophy is surprisingly simple and beautiful. Let's break down .net app into its building blocks. There's request, which is being processed and there's a response. How to represent request and response in the simplest way? There's something under HEADERS "label" and something under BODY "label" on the request side, and something under COOKIES "label" and something under BODY "label" on the response side. So `IDictionary<string, object>` seems to be a pretty good abstraction, doesn't it? Which leads to the conclusion that building block for app processing pipeline is `Func<IDictionary<string, object>, Task>` - _something happens based on data_. OWIN people like to call it ___AppFunc___ for readability and often create alias in the code. Last definition: middleware is `Func<AppFunc, AppFunc>` - block that can call next block.

Now let's see how the theory looks like in practice. My poor man's MVC is as follows:

{% highlight C# %}
public class PoorMansMvc
{
    private Func<IDictionary<string, object>, Task> next;

    public PoorMansMvc(Func<IDictionary<string, object>, Task> next)
    {
        this.next = next;
    }

    public async Task Invoke(IDictionary<string, object> context)
    {
        var responseStream = context["owin.ResponseBody"] as Stream;
        using (var responseWriter = new StreamWriter(responseStream))
        {
            await responseWriter.WriteAsync("I am poor man's MVC");
        }

        await this.next.Invoke(context);
    }
}
{% endhighlight %}

`owin.ResponseBody` is where, by OWIN specification, the content of the response should be placed.

The driver for entire process here is __Katana__: OWIN implementation by Microsoft. Katana packages can be found on nuget. You'll need `Microsoft.Owin.Host.HttpListener` and `Microsoft.Owin.Hosting` to run this sample. The entire C# console app that self-hosts the poor man's MVC looks like this (notice _AppFunc_ alias for readability):

{% highlight C# %}
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading.Tasks;
using Microsoft.Owin.Hosting;
using Owin;

namespace KatanaVNextSample
{
    using AppFunc = Func<IDictionary<string, object>, Task>;

    class Program
    {
        static void Main(string[] args)
        {
            WebApp.Start<Startup>("http://localhost:8002");
            Console.ReadKey();
        }
    }

    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            app.Use<PoorMansMvc>();
        }
    }

    public class PoorMansMvc
    {
        private AppFunc next;

        public PoorMansMvc(AppFunc next)
        {
            this.next = next;
        }

        public async Task Invoke(IDictionary<string, object> context)
        {
            var responseStream = context["owin.ResponseBody"] as Stream;
            using (var responseWriter = new StreamWriter(responseStream))
            {
                await responseWriter.WriteAsync("I am poor man's MVC");
            }

            await this.next.Invoke(context);
        }
    }
}
{% endhighlight %}

No magic here, everything is self-explanatory, you just need to know that `WebApp` will try to run `Configuration()` method for the given class and that's where you want to chain your middlewares.

One last step to mimic vNext is pure cosmetics. With the following extension method:

{% highlight C# %}
public static class AppBuilderExtensions
{
    public static IAppBuilder UseMvc(this IAppBuilder appBuilder)
    {
        return appBuilder.Use<PoorMansMvc>();
    }
}
{% endhighlight %}

you can use `UseMvc()` method in `Configuration()` method and feel like Microsoft .NET MVC team member.

Hopefully this article have helped you understand the _magic_ of new ASP.NET vNext configuration methods and it should also be a good start before learning how middlewares can be chained, and other more advanced OWIN stuff.
