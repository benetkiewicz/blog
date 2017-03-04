---
layout: post
title: "Azure AD Authorization"
date: "2017-02-14 21:00"
---

In the third part of this Azure AD series I will cover various features that can be utilized to implement authorization in your application. Azure supports groups and roles which are easilly transformable to ASP.NET Identity claims. There is also OAuth2 client credentials flow available to utilize in machine to machine scenarios. If there are very refined rules driving your authorization, you can utilize Azure Graph API to get access to almost any data in directory's possession in order to make your security desisions.

### Groups in B2E

Groups are top level active directory objects, that can contain any number of users. Groups in B2E can also be nested, which unfortunately is not the case for B2C. Nesting groups can lead to some issues when using Graph API to check membership. Some Graph API calls are transitive and some or not, so asking about assignment to a group high in the tree hierarchy can give different results based on the api call of choice.

You can manage groups and user assignments in both legacy and new portals or using Graph API. In order to have groups automatically set as claims for ASP.NET identity, you need to flip the switch in application manifest file, because group claims are not being returned by default. Go to application settings in legacy Azure portal, select _Manage Manifest_, download file, find _groupMembershipClaims_ and change _null_ value to _SecurityGroup_. The other permitted value is _All_ but this will return all user's distribution lists, which for some users, in some corporations, can easilly excess the limit, which for JWT token is set to 200. As a side note: in new portal, you can just edit manifest in the browser, which is kind of nice.

If all goes well, ASP.NET Identity will contain claims with IDs of all groups user is assigned to:

![Azure principal claims groups]({{ site.url }}images/azure_principal_claims_groups.png)

### Groups in B2C

This is all good and fine in B2E scenarios. B2C will not return groups in payload, no matter what configuration voodoo we will do. Graph API is a way to go and fortunatelly is not that hard to play with.

To query B2C Active Directory using Graph API you need a special __service application__, other than the one you are currently working with. Set of powershell commands required to create this azure app is described in detail in [MSDN documentation][msdndoc]. Once you have that, the following code will get the list of IDs of all groups user is assigned to:

[msdndoc]: https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-devquickstarts-graph-dotnet#register-a-service-application-in-your-tenant

```csharp
var credential = new ClientCredential(clientId, clientSecret);
AuthenticationResult result = await authContext.AcquireTokenAsync("https://graph.windows.net/", credential);

string userObjectId = ctx.Identity.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value;
string api = "/users/" + objectId + "/getMemberGroups";
string url = "https://graph.windows.net/" + tenant + api + "?api-version=1.6";
var request = new HttpRequestMessage(HttpMethod.Post, url);
request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);
request.Content = new StringContent("{securityEnabledOnly: false}", Encoding.UTF8, "application/json");

var http = new HttpClient();
var response = await http.SendAsync(request);
return await response.Content.ReadAsStringAsync();
```

The above code requires some explanation. The goal is to make a _POST_ request to Graph _getMemberGroups_ operation, parametrized with user's _objectId_ - user's unique identifier in AD. Call is protected with Bearer token, which needs to be provided in authentication header of the request. Token is obtained from Azure with use of this special credential set I mentioned in a previous paragraph. Notice also a _securityEnabledOnly_ parameter provided in request body which is rough equivalent of _groupMembershipClaims_ setting from json app manifest.

Please also note that you may not find _objectidentifier_ claim on your user's identity. It depends on B2C policy settings, so head out to new portal and make sure that _Policies_ -> _policy name_ -> _Edit_ -> _Application claims_, and select _User's Object ID_, see below:

![Azure AD B2C objectid claim]({{ site.url }}images/azure_ad_b2c_objectidclaim.png)

### Roles

There are some predefined roles in azure, which you can see by going to user detail tab in legacy portal. You can also define your custom roles per application scope. Roles setup is, again, tedious json edit in application manifest file. Go to your application download and edit manifest, add the following snippet in the empty roles section (adjust to your needs):

```javascript
{
    "appRoles": [
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "Reader",
      "id": "d2c2ade8-98f8-45fd-aa4a-6d06b947c64f",
      "isEnabled": true,
      "description": "Readers Have the ability to create tasks.",
      "value": "Reader"
    },
    {
      "allowedMemberTypes": [
        "User"
      ],
      "displayName": "Writer",
      "id": "d2c2ade8-98f8-45fd-aa4a-6d06b947c65f",
      "isEnabled": true,
      "description": "Writers Have the ability to create tasks.",
      "value": "Writer"
    }
  ]
}
```

You can set user in role while assigning him to an app. In legacy portal go to app -> assign and the roles dropdown should appear automatically. Remember that if you have only one role for your app, you will not be presented with any choice and assigned users will automaticaly have this role assigned. Roles will automatically be translated to claims in ASP.NET Identity. (TODO: screenshot)

I haven't found a way to assign user to a role in new portal.

TODO: fourth post
### M2M

* manifest
* permissions (screenshot?)
* consume c#
