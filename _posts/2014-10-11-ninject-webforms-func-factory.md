---
layout: post
title:  "Ninject in WebForms application"
date:   2014-10-11 16:57:59
categories: csharp ninject
---

Recently I tried to set up ninject into WebForms application, like that:

{% highlight C# %}
public class Global : NinjectHttpApplication
{
    protected override IKernel CreateKernel()
    {
        // configure here
        return new StandardKernel();
    }
}
{% endhighlight %}

but the app has been throwning exceptions on start:

    [NullReferenceException: Object reference not set to an instance of an object.]
    System.Web.HttpApplication.RegisterEventSubscriptionsWithIIS(IntPtr appContext, HttpContext context, MethodInfo[] handlers) +209
    System.Web.HttpApplication.InitSpecial(HttpApplicationState state, MethodInfo[] handlers, IntPtr appContext, HttpContext context) +172
    System.Web.HttpApplicationFactory.GetSpecialApplicationInstance(IntPtr appContext, HttpContext context) +336
    System.Web.Hosting.PipelineRuntime.InitializeApplication(IntPtr appContext) +296

    [HttpException (0x80004005): Object reference not set to an instance of an object.]
    System.Web.HttpRuntime.FirstRequestInit(HttpContext context) +9930508
    System.Web.HttpRuntime.EnsureFirstRequestInit(HttpContext context) +101
    System.Web.HttpRuntime.ProcessRequestNotificationPrivate(IIS7WorkerRequest wr, HttpContext context) +254

It appears that the code from above is valid only for Asp.Net MVC apps, for WebForms you should install `ninject.web` nuget package, which registers some `App_Start/` classes via `WebActivator`.

Another thing worth to remember is that WebForms require parameterless constructors for its page classes, so you need to inject dependecies via properties using `[Inject]` attribute.

###Bonus
One little peculiarity of ninject that got me stumbled for few minutes are `Func<T>` factory methods (_ninject.extensions.factory_). I'll just steal an example code from project [website]
{% highlight C# %}
class Foo
{
    readonly Func<Bar> barFactory;

    public Foo(Func<Bar> barFactory)
    {
        this.barFactory = barFactory;
    }

    public void Do()
    {
        var bar = this.barFactory();
        ...
    }
}
{% endhighlight %}

The above code will work without providing any binding configuration for `Func<Bar>`. This is quick (but smelly, you have to admit) way of creating factories for simple entities that don't take any arguments.

[website]: https://github.com/ninject/ninject.extensions.factory/wiki/Func
