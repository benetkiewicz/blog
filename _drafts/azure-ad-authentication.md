---
layout: post
title: "Azure AD Authentication"
date: "2016-12-21 09:59"
---
This is the first post in a series about Azure Active Directory. I want to cover the following topics:
* Authentication in Azure Active Directory (Business to Enterprise)
* Authentication in Azure AD B2C  (Business to Consumer)
* Authorization with Azure AD
* Working with Azure AD using Graph API

Azure Active Directory (in this post I am referring to old Business to Enterprise type) is a cloud service that puts a synchronized mirror of On-Premise Active Directory in Azure Cloud and provides some neat APIs to work with it. On of the main reasons why Azure AD came to existence was to provide an easy licensing model for Office and other Enterprise tools in the cloud. The logic goes like that: enterprises use Office, enterprises use also Active Directory (On-Premise). Office 365 lives in a cloud, so let's move AD to the cloud, so enterprise can use their AD accounts for Office.

What's in that for us - developers? With AD we have a reliable and up-to-date user store which, with Azure AD boost, can be used for internet apps without any additional infrastructure. There are pretty good chances that if a company uses Office 365, you're almost set to get all the goodies from Azure AD. Moreover MSDN subscription licenses also have Azure for developers to play with and Azure AD is just one of services that can be used with that subscription. This is exactly what I'll be using in my demo.

Let's see how we can implement a simple authentication in ASP.NET MVC internet application.
First, let's have a look at some basic concepts in Azure AD:
* azure application - it is a handler for your actual app in azure. It is an entity which holds information on how your web application interacts with azure, how azure should interact with your app and many other configuration parameters. Internally represented by _Client Id_
* Client Id is a GUID which uniquely identifies Azure app
* Tenant is a... tenant. Think of it a specific instance of AD (in case I described above - the synced On-Premise one) in Azure Active Directory service. It is represented by a name, but more importantly, it has a default domain, which was configured while creating a tenant, and will be used in our web app configuration.
* Open ID Connect - Authentication protocol, built on top of OAuth2, used in Azure AD authentication scenarios.

Now we'll actually build stuff. Starting in Azure portal, we need to create an application and obtain some basic configuration parameters for our ASP.NET MVC app.
1. In Azure portal go to Active Directory, Default Directory, Application tab.
2. Click Add at the bottom.
3. Click _Add an application my organization is developing_. It means that you have a new web application that you want to integrate with Azure. There are already tons of other federated services that can be protected with Azure AD and the list is under _Gallery_ link in that dialog window.
4. In the next step, provide the descriptive name. Choose _Web application and/or web API_ option. _Native client application_ option is useful in different scenarios. The main difference is underlying OAuth2 flow.
5. Provide _Sign-On URL_ and _App ID URI_. _Sign-On URL_ setting will be used in default application configuration as a _reply to_ URL. _App ID URI_ is like a namespace of your app. The URI must be in a verified custom domain for an external user to grant your app access to their data in Microsoft Azure AD, but we will not use this app in app-to-app resource authorization scenario. Any unique URI will be just fine.
6. When you're done with the wizard, you will see azure app summary screen, where most of settings can be changed or adjusted and where you can see _Client ID_ for your app.

Right, we're done on Azure Portal side, now switch to the code.

```
Install-Package Microsoft.Owin.Security.OpenIdConnect
Install-Package Microsoft.Owin.Security.Cookies
Install-Package Microsoft.Owin.Host.Systemweb
```
Third package for Startup.cs.
You need to have Azure app _Reply Url_ in sync with URL where you actually host your app. After successful user authentication in Azure Portal, Azure will redirect back to the given _Reply Url_. So if you create your sample app in Visual Studio and host it in IIS Express, make sure your localhost:port URL will be present in Azure app config.  http://localhost:44404/
