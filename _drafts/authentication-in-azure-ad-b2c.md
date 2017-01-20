---
layout: post
title: "Authentication in Azure AD B2C"
date: "2017-01-08 12:19"
categories: azure active directory csharp
---

## The tale of two portals

In the previous part of this series we learned how to incorporate simple authentication with Azure AD (B2E) into vanilla ASP.NET MVC App. As a prerequisite, we had to create a bunch of entities in Azure Portal (https://manage.windowsazure.com). In this blog post I will show how to achieve a similar thing but with Business to Consumer (B2C) falovour of Azure Active Directory and we will learn what are some additional benefits of using B2C.

This is also a moment where new Azure portal needs to be introduced: (https://portal.windowsazure.com). There are several entry points to this new management tool, one of them is a banner that will bug you in "classic" Azure portal, but please note that if you wan't to play with B2C you must use the new portal because "classic" one is simply unaware of most B2C features and configuration options.

B2C is a different type of Azure Active Directory, which is a fact that you state while creating it. In legacy portal click _NEW_, _App Services_, _Active Directory_, _Directory_, _Custom Create_. In Add Directory dialog remember to select _This is a B2C directory_ checkbox. Interestingly enough, you cannot add a B2C AD from withing new portal because at the time writing this (12/1/2017) Microsoft did not finigh adding support for that in new portal. Things essential to B2C AD configuration are supported though and not available in legacy portal. You just can't create a directory here. Madness!

So, once you create a B2C directory from withing legacy portal, you can use preffered (legacy or new) UI to create app and user in your newly created directory. This process is pretty much the same as for B2E, so I will not cover the details here. Once setup is done, you need _Client Id_ and _Tenant_ to replace values in a configuration in our B2E example and sign in process should work the same. Remember to use an account from B2E directory!

We've got to the point where we're stuck again with a pre-defined set of users, no sign-up, no federation with other identity providers, nothing fancy. But this is where it gets interesting. We need to create and incorporate a _policy_. Head to the old portal, navigate to a landing page for your B2C directory and click _Manage B2C settings_ link. It will redirect you to the new portal (told'ya - madness!) and open _AZURE AD B2C SETTINGS_ section automatically. Let's see how we can set up our application with an onboarding process for users from internet.

Go to _Sign-up policies_, click _Add_, give it a name (I named it _blog_policy_ so the full name will be _B2C_1_blog_policy_) and in _Identity providers_ section select _Email signup_. It is the only build-in azure sign up policy which also has an advantage of not having any additional configuration. For users it will be as simple as it can be - their new accounts will be associated with their emails, they will have to create a password and it will essentially create a new account in our B2C directory.

Now when you click this policy in portal UI you'll get a nice simple UI that will help you test the policy.
![Azure AD test policy]({{ site.url }}images/azure_ad_test_policy.png)

Notice that _Run now_ button navigates to the following URL:

```
https://login.microsoftonline.com/aisappengine.onmicrosoft.com/oauth2/authorize?p=B2C_1_blog_policy&client_Id=09866094-b889-46f4-8903-1e66799518df&nonce=defaultNonce&redirect_uri=http%3A%2F%2Flocalhost%3A44404%2F&scope=openid&response_type=id_token&prompt=login
```
It is pretty much the same as a redirect URL in B2E scenario but there's a `p` (as in policy) query string parameter which is triggering proper behavior on Azure side. Our whole programming task right now is to convince MVC to include proper policy name when automatically redirecting to Azure when it encounters `Authorize` attribute.

```powershell
Update-package Microsoft.IdentityModel.Protocol.Extensions
```

![azure b2c claims]({{ site.url }}images/azure_b2c_claims.png)

```csharp
TokenValidationParameters = new TokenValidationParameters
{
    NameClaimType = "name",
    SaveSigninToken = true //important to save the token in boostrapcontext
}
```

Setup _Identity Providers_ for federation.
