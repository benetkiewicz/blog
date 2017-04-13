---
layout: post
title: "Machine to machine authorization"
date: "2017-04-13 20:41"
---

This is the last post in the series about Azure Active Directory authentication and authorization. To navigate previous post, navigate to the following links:

* first
* Second
* third

## Business case

In this post I will cover how to protect access to Web API with baerer tokens, using Azure Active Directory. The business case is as follows:

* My core business app gathers users data in a very structured way. It is being used by Organizations, that create Forms for their users to fill, and each filled and completed Form becomes a Submission.

* My API is a reporting service, serving data in a way that follows the structure outlined above.

```csharp
[HttpGet]
[Route("api/v1/{organization}/{form}")]
public AppResponse Get(string organization, string form, string dateFrom)
```

* I need to be able to grant access to my API to various third party services for their integration purposes. Security must be quite granular, I need to be able to grant access to all submissions of the form, all form submissions in a given organization or to all data.

So for example, there can be an app which should be able to query:

    GET http://foo.com/api/v1/awesomecorp/registration?dateFrom=2017-01-01

but should fail with _403_ when accessing

    GET http://foo.com/api/v1/awesomecorp/supersurvey?dateFrom=2017-02-02

## Azure setup

As usual, I need Azure AD app configuration for my API. Next, edit application manifest and add some roles. This time they will be _application roles_ as opposed to _user roles_ which we added in part TODO!

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
Next, we design an onboarding process. Each partner that wants access to our API needs to have Azure AD created. In newly create app in azure portal, go to _CONFIGURE_ and click big green _Add application_ button in the bottom. Search for Azure app that represents your API and click plus sign in the grid next to it.
![Azure App Permissions](images/azure_app_permissions_listing.png)
Back in main settings panel, in _permissions to other applications_ section at the bottom, you will have a new row representing our API, with all _Application Permissions_ it provides. Select proper items from the list and save.
![Azure App Selection](images/azure_app_permissions_selection.png)
