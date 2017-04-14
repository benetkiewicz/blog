---
layout: post
title: Machine to machine authorization in Azure AD
date: '2017-04-14 15:23'
categories: azure active directory csharp
---

This is the last post in the series about Azure Active Directory authentication and authorization. To see previous posts, navigate to the following links:

* [Deep dive into Azure AD Authentication][first]
* [Authentication in Azure AD B2C][second]
* [Azure AD Authorization][third]

[first]: {{site.url}}azure/active/directory/csharp/2016/12/30/azure-ad-authentication.html
[second]: {{site.url}}azure/active/directory/csharp/2017/02/05/authentication-in-azure-ad-b2c.html
[third]: {{site.url}}azure/active/directory/csharp/2017/03/05/azure-ad-authorization.html

## Business case

In this post I will cover how to protect access to Web API with bearer tokens, using Azure Active Directory. The business case is as follows:

* My core business app gathers users data in a very structured way. It is being used by Organizations, that create Forms for their users to fill, and each filled and completed Form becomes a Submission.

* My API is a reporting service, serving data in a way that follows the structure outlined above.

```csharp
[HttpGet]
[Route("api/v1/{organization}/{form}")]
public AppResponse Get(string organization, string form, [FromUri]string dateFrom)
```

* I need to be able to grant access to my API to various third party services for their integration purposes. Security must be quite granular, I need to be able to grant access to all submissions of the form, all form submissions in a given organization or to all data.

So for example, there can be an app which should be able to query:

    GET http://foo.com/api/v1/awesomecorp/registration?dateFrom=2017-01-01

but should fail with _403_ when accessing

    GET http://foo.com/api/v1/awesomecorp/supersurvey?dateFrom=2017-02-02

## Azure setup

As usual, I need Azure AD app configuration for my API. Next, edit application manifest and add some roles. This time they will be _Application_ roles as opposed to _User_ roles which we added in previous post in the series.

```javascript
{
  "appId": "067f5d6c-a79f-4430-bb82-00aa060432b1",
  "appRoles": [
    {
      "allowedMemberTypes": [
        "Application"
      ],
      "description": "Allow the application to access ALL APIs",
      "displayName": "Access ALL APIs",
      "id": "0c65e07c-9d03-4617-83d5-09adae44c5e3",
      "isEnabled": true,
      "value": "/api/v1"
    },
    {
      "allowedMemberTypes": [
        "Application"
      ],
      "description": "Allow the application to access Supercorp's Registration",
      "displayName": "Access Supercorp's Registration",
      "id": "0c65e07c-9d03-4617-83d3-09adae44c4e2",
      "isEnabled": true,
      "value": "/api/v1/awesomecorp/registration"
    }
}
```

Notice `allowedMemberTypes` being set to `Application` and the value of the role.
Next, we design an onboarding process. Each partner that wants access to our API needs to have Azure AD application created. In newly create app in azure portal, go to _CONFIGURE_ and click big green _Add application_ button in the bottom. Search for Azure app that represents your API and click plus sign in the grid next to it.

![Azure App Permissions]({{ site.url }}images/azure_app_permissions_listing.png)

Back in main settings panel, in _permissions to other applications_ section at the bottom, you will have a new row representing our API, with all _Application Permissions_ it provides. Select proper items from the list and save.

![Azure App Selection]({{ site.url }}images/azure_app_permissions_selection.png)

### API code adjustments

First, API needs to be instructed where to expect incoming Auth data and how to ensure they aren't spoofed. The following code in `Startup.js` is required:

```csharp
public void Configuration(IAppBuilder app)
{
    app.UseWindowsAzureActiveDirectoryBearerAuthentication(
     new WindowsAzureActiveDirectoryBearerAuthenticationOptions
     {
         TokenValidationParameters = new System.IdentityModel.Tokens.TokenValidationParameters
         {
             ValidAudience = ConfigurationManager.AppSettings["ida:Audience"]
         },
         Tenant = ConfigurationManager.AppSettings["ida:Tenant"]
     });
}     
```

_ida:Audience_ is _APP ID URI_ setting configured in Azure portal and it ensures that the token is meant for our API. _ida:Tenant_ is what we've had in all previous code examples.

Then it all comes down to analyzing roles embedded within the incoming request token. If requested API route starts with some role value then request is authorized to proceed.

```csharp
public class AuthorizeApiAzureAttribute : AuthorizeAttribute
{
    protected override void HandleUnauthorizedRequest(HttpActionContext actionContext)
    { /* omitted for clarity */ }

    protected override bool IsAuthorized(HttpActionContext actionContext)
    {
        var claims = ClaimsPrincipal.Current.FindAll(c => c.Type == ClaimTypes.Role || c.Type == "roles");
        if (claims == null || !claims.Any())
        {
            return false;
        }

        List<string> roleValues = claims.Select(c => c.Value).ToList();
        var url = actionContext.Request.RequestUri;
        return IsAuthorizedInternal(url, roleValues);
    }

    protected static bool IsAuthorizedInternal(Uri url, List<string> roleValues)
    {
        string path = String.Format(@"{0}{1}{2}{3}", url.Scheme, Uri.SchemeDelimiter, url.Authority, url.AbsolutePath);
        string apiPath = path.Substring(path.IndexOf(@"/api"));
        return roleValues.Any(rv => apiPath.StartsWith(rv, StringComparison.InvariantCultureIgnoreCase));
    }
}
```

### What about client code?

Please note that partner's clients are not bound to CSharp or .NET in general. It is cross-platform standard, so all they need to do is to obtain a token from azure using their _ClientId_ and _Secret_ and use this token with an API request. The following code should be very easy to port to other programming languages:

```csharp
var httpClient = new HttpClient();
var tokenRequestMessage = new HttpRequestMessage(HttpMethod.Post, TokenEndpointUrl);
HttpContent tokenRequestMsg = new FormUrlEncodedContent(new[]
{
    new KeyValuePair<string, string>("grant_type", "client_credentials"),
    new KeyValuePair<string, string>("client_id", clientId),
    new KeyValuePair<string, string>("client_secret", secret),
    new KeyValuePair<string, string>("resource", resource)
});
tokenRequestMessage.Content = tokenRequestMsg;

var tokenResponse = httpClient.SendAsync(tokenRequestMessage).Result;
string rawTokenResponse = tokenResponse.Content.ReadAsStringAsync().Result;
TokenRequestResponse deserializedTokenResponse = JsonConvert.DeserializeObject<TokenRequestResponse>(rawTokenResponse);

var apiClient = new HttpClient();
apiClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", deserializedTokenResponse.access_token);
var apiResponse = apiClient.GetAsync("http://foo.com/api/v1/xml/awesomecorp/registration?dateFrom=2017-01-01").Result;
string content = apiResponse.Content.ReadAsStringAsync().Result;
```

In the code above, _clientId_ and _secret_ are as set in client's app in Azure Portal and _resource_ matches _ida:Audience_ in API.
