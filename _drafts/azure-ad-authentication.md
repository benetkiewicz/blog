---
layout: post
title: "Deep dive into Azure AD Authentication"
date: "2016-12-30 21:59"
categories: azure active directory csharp
---

## Azure AD introduction

This is the first post in a series about Azure Active Directory. I want to cover the following topics:

* Authentication in Azure Active Directory (Business to Enterprise)
* Authentication in Azure AD B2C  (Business to Consumer)
* Authorization with Azure AD
* Working with Azure AD using Graph API

Azure Active Directory (in this post I am referring to old Business to Enterprise type) is a cloud service that puts a synchronized mirror of On-Premise Active Directory in Azure Cloud and provides some neat APIs to work with it. On of the main reasons why Azure AD came to existence was to provide an easy licensing model for Office and other Enterprise tools in the cloud. The logic goes like that: enterprises use Office, enterprises also use Active Directory (On-Premise). Office 365 lives in a cloud, so let's move AD to the cloud, so enterprise can use their AD accounts for Office.

![Azure AD Architecture]({{ site.url }}images/azure_ad_architecture.png)

What's in that for us - developers? With AD we have a reliable and up-to-date user store which, with Azure boost, can be used for internet apps without any additional infrastructure. There are pretty good chances that if a company uses Office 365, you're almost set to get all the goodies from Azure AD. Moreover MSDN subscription licenses also have Azure for developers to play with and Azure AD is just one of services that can be used with that subscription. This is exactly what I'll be using in my demo.

## Azure AD basic concepts and portal overview

Let's see how we can implement a simple authentication in ASP.NET MVC internet application.
First, let's have a look at some basic concepts in Azure AD:

* azure application - it is a handler for your actual app in azure. It is an entity which holds information on how your web application interacts with azure, how azure should interact with your app and many other configuration parameters. Internally represented by _Client Id_
* Client Id is a GUID which uniquely identifies Azure app
* Tenant is a... tenant. Think of it a specific instance of AD (in case I described above - the synced On-Premise one) in Azure Active Directory service. It is represented by a name, but more importantly, it has a default domain, which was configured while creating a tenant, and will be used in our web app configuration.
* Open ID Connect - Authentication protocol, built on top of OAuth2, used in Azure AD authentication scenarios.

Now we'll actually build stuff. Starting in Azure portal, we need to create an application and obtain some basic configuration parameters for our ASP.NET MVC app.

1. In Azure portal go to _Active Directory_, _Default Directory_, _Application_ tab.
2. Click _Add_ at the bottom.
3. Click _Add an application my organization is developing_. It means that you have a new web application that you want to integrate with Azure. There are already tons of other federated services that can be protected with Azure AD and the list is under _Gallery_ link in that dialog window.
4. In the next step, provide the descriptive name. Choose _Web application and/or web API_ option. _Native client application_ option is useful in different scenarios. The main difference is underlying OAuth2 flow.
5. Provide _Sign-On URL_ and _App ID URI_. _Sign-On URL_ setting will be used in default application configuration as a _reply to_ URL. _App ID URI_ is like a namespace of your app. In most cases any unique URI will be just fine. The URI must be in a verified custom domain for an external user to grant your app access to their data in Microsoft Azure AD, but we will not use this app in app-to-app resource authorization scenario.
6. When you're done with the wizard, you will see azure app summary screen, where most of settings can be changed or adjusted and where you can see _Client ID_ for your app.

## Programming Azure AD APIs

Right, we're done on Azure Portal side, now switch to the code.

Create an empty MVC app in Visual Studio and install the following Nuget packages:

```powershell
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.Systemweb
```

First two packages are directly related to Azure, third package will allow you to wire it all up in Startup.cs.

You need to have Azure app _Reply Url_ in sync with URL where you actually host your app. After successful user authentication in Azure Portal, Azure will redirect back to the given _Reply Url_. So if you create your sample app in Visual Studio and host it in IIS Express, make sure your localhost:port URL will be present in Azure app config.

Now head to `Startup.cs` class and add the following lines:

```csharp
public void Configuration(IAppBuilder app)
{
    // ...
    ConfigureAuth(app);
}

private void ConfigureAuth(IAppBuilder app)
{
    app.UseCookieAuthentication(new CookieAuthenticationOptions());
    string authorityUrl = string.Format("https://login.microsoftonline.com/{0}", WebConfigurationManager.AppSettings["ida:Tenant"]);
    app.UseOpenIdConnectAuthentication(new OpenIdConnectAuthenticationOptions
        {
            Authority = authorityUrl,
            ClientId = WebConfigurationManager.AppSettings["ida:ClientId"]
        });
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);
}
```

There are couple of things that need explanation. First, `app.UseCookieAuthentication()` line says that we will have cookie based auth session. Important thing is that this middleware needs to be higher in a pipeline than OpenIdConnect middleware. OpenIdConnect doesn't have any session persistence on its own, so even if Azure authentication would work, without proper cookie management on application side, user still would look like anonymous and we would get infinite redirect loop.
Second, we configure some core settings for OpenIdConnect. Authority URL is where all metadata manifests, sign-on/sign-out endpoints are, all that protocol good stuff. OpenIdConnect library needs only the root URL, it will figure out the rest automatically. If you're interested what are all these URLs related to your app, you can see them in azure portal. Go to your app definition in Azure portal and click _View Endpoints_ button in the bottom.![Azure View Endpoints]({{ site.url }}images/azure_endpoints.png)

There are multiple ways of building authority URLs. As you can see Azure portals builds them with guid/object id convention. The same result can be achieved by using tenant domain identifier as in the code. It's just easier to obtain, clearly visible in azure portal. So because my tenant _piotrais.onmicrosoft.com_ object id is _96a4c14c-38d2-481b-931e-59502b6a22a1_, the following authority URLs are in fact pointing to the same thing:

* _https://login.microsoftonline.com/96a4c14c-38d2-481b-931e-59502b6a22a1/_
* _https://login.microsoftonline.com/piotrais.onmicrosoft.com/_

There is a third convention for building authority URLs, for multi-tenant apps it is always _https://login.microsoftonline.com/common/_.

Enough explanation! We're done! Now putting `[Authorize]` attribute on a `HomeController` class will make it a protected asset, where anonymous users don't have access. Moreover, it will automatically trigger a redirect to Azure login page where user will be able to sign in. Azure will also take care of redirecting back to our app and OpenIdConnect middleware we just configured in `Startup.cs` will consume incoming payload, set proper cookies and identity.

So running the following code and signing in as demo@piotrais.onmicrosoft.com

```csharp
[Authorize]
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return Content(string.Format("Hello {0}", User.Identity.Name));
    }
}
```

will give the following result: `Hello demo@piotrais.onmicrosoft.com`

## A word about users and summary

Speaking about users... Where do we get users for sign in? In enterprise scenarios there will be plenty of users from synced On-Premise AD. In development scenario, we need to create a user in Azure portal. In your Default Directory go to _Users_ tab, click _Add User_ button at the bottom, provide a user name and you're done. This is useful for testing scenarios but I can also imagine an app with small, constant number of users or an app with admistrative module, where this authentication/authorization scheme would be just enough.

Azure AD also supports user onboarding process with some self-service and even federation to popular identity providers like google, facebook or twitter. It is Azure AD B2E younger sibling called Azure AD Business to Consumer (B2C). I will cover this topic in the next part of this series.
