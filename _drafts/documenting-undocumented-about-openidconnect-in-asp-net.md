---
layout: post
title: "Documenting undocumented about OpenIDConnect in ASP.NET"
date: "2016-01-29 22:54"
---

This post could be titled _Little known facts about OpenIDConnect in ASP.NET_ but is really about things that aren't emphasized in samples, are not so well documented or just gave me some new gray heir while I implemented my piece of code.

Let's get straight to the code and discuss interesting lines after. Typically there are two interesting blocks of code, one defining protocol specific configuration for our project, like:
- ClientID - our app
- Authority - STS
- Redirect URL
- STS callbacks

{% highlight C# %}
 app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
        UseTokenLifetime = false,
        ClientId = ConfigHelper.ClientId,
        Authority = ConfigHelper.Authority,
        PostLogoutRedirectUri = ConfigHelper.PostLogoutRedirectUri,
        Notifications = new OpenIdConnectAuthenticationNotifications
        {
            AuthorizationCodeReceived = context =>
            {
                ClientCredential credential = new ClientCredential(ConfigHelper.ClientId, ConfigHelper.AppKey);
                string userObjectId = context.AuthenticationTicket.Identity.FindFirst(Globals.ObjectIdClaimType).Value;
                AuthenticationContext authContext = new AuthenticationContext(ConfigHelper.Authority);
                AuthenticationResult result = authContext.AcquireTokenByAuthorizationCode(context.Code, new Uri(ConfigHelper.PostLogoutRedirectUri), credential, ConfigHelper.GraphResourceId);
                return Task.FromResult(0);
            },
            AuthenticationFailed = context =>
            {
                logger.Error("auth failed", context.Exception);
                context.HandleResponse();
                context.Response.Redirect("/Error");
                return Task.FromResult(0);
            }
        }
    });
{% endhighlight %}

Second section is about maintaining user session and user identity being result of successful (or not) authentication.

{% highlight C# %}
double sessionTimeout = 15.0;
long sessionTimeEnd = 0;

app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

app.UseCookieAuthentication(new CookieAuthenticationOptions
{
    CookieManager = new SystemWebCookieManager(),
    CookieName = "MyCookie_Auth",
    Provider = new CookieAuthenticationProvider
    {
        OnResponseSignIn = ctx =>
        {
            sessionTimeEnd = ctx.Options.SystemClock.UtcNow.AddMinutes(sessionTimeout).UtcTicks;

            ClaimsIdentity claimsId = ctx.Identity;
            List<string> objectIds = ClaimHelper.GetGroups(claimsId);
            var myGroups = new List<Group>();
            var myDirectoryRoles = new List<DirectoryRole>();
            string userObjectId = ctx.Identity.FindFirst(Globals.ObjectIdClaimType).Value;
            Task.Run(async () => { await GraphHelper.GetDirectoryObjects(objectIds, myGroups, myDirectoryRoles, userObjectId); }).Wait();
            foreach (var myGroup in myGroups)
            {
                ctx.Identity.AddClaim(new Claim(ClaimTypes.Role, myGroup.DisplayName.ToLower(), ClaimValueTypes.String));
            }
        },
        OnValidateIdentity = ctx =>
        {
            bool reject = true;

            long currentTick = ctx.Options.SystemClock.UtcNow.UtcTicks;
            if (sessionTimeEnd == 0)
            {
                sessionTimeEnd = ctx.Options.SystemClock.UtcNow.AddMinutes(sessionTimeout).UtcTicks;
            }

            reject = currentTick > sessionTimeEnd;
            if (!reject)
            {
                double sessionTimeRemainingInMinutes = TimeSpan.FromTicks(sessionTimeEnd - currentTick).TotalMinutes;
                if (sessionTimeRemainingInMinutes < sessionTimeout/2 && sessionTimeRemainingInMinutes > 0)
                {
                    sessionTimeEnd = currentTick + TimeSpan.TicksPerMinute*Convert.ToInt64(sessionTimeout);
                }
            }

            if (reject)
            {
                ctx.RejectIdentity();
                ctx.OwinContext.Authentication.SignOut(
                    new AuthenticationProperties {RedirectUri = ConfigHelper.PostLogoutRedirectUri},
                    OpenIdConnectAuthenticationDefaults.AuthenticationType,
                    CookieAuthenticationDefaults.AuthenticationType
                    );
            }

            return Task.FromResult(0);
        }
    }
});
{% endhighlight %}
