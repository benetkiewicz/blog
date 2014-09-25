---
layout: post
title:  "ASP.NET bundles usefull in angularjs development"
date:   2014-09-20 16:57:59
categories: angularjs
---

__Disclaimer__: I know that there are tools like Grunt, Bower, etc. and they are first choice for this job for front-end developers. I am trying to find easiest way to join asp.net and angularjs forces and the following hint will work flawlessly in angularjs + .net backend scenario, so why not use it?

One of the biggest annoyances when you work on angularjs project and you follow convention for file separation, is when you forget to include newly created angular component (controller, directive, service, etc.) to you master script/page and it just won't work. It would be great if each and every of your scripts could be automatically added to your SPA app, and all of its features would be immediately ready to test.

[Bundles] to the rescue! The following snippet in your bundle configuration will do the trick:

{% highlight C# %}
public static void RegisterBundles(BundleCollection bundles)
{
    var b = new Bundle("~/scripts/application").IncludeDirectory("~/scripts/app", "*.js", true)
    bundles.Add(b);
}
{% endhighlight %}

It will create a `scripts\application` virtual resource that will contain all javascript files in your app folder and all nested folders. Note that if you prefer to store routing templates in one of the `app` subfolders, they will not be included into bundle, so you're safe here. You need to include that bundle on your web page, just as any regular script:

{% highlight html %}
<script src="scripts/application" type="text/javascript"></script>
{% endhighlight %}

From now on you can just write code, add new files, start the app and everything will be there. Additional benefit will be limited number of http requests required to download content of your website.

[Bundles]: http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification
