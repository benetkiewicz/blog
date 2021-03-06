---
layout: post
title:  "Server side valdidation for angularjs SPA"
date:   2014-09-28 16:57:59
categories: angularjs asp.net
---

Consider the following view model classes:

{% highlight C# %}
public class Registration
{
    // some properties ommited for clarity
    public CarManufacturer CarManufacturer { get; set; }
}
{% endhighlight %}

{% highlight C# %}
public class CarManufacturer
{
    public string Name { get; set; }

    [RegularExpression("^(BU|FD|FT|WV)$")]
    public string Id { get; set; }
}
{% endhighlight %}

The following server side validation code will create json error response that is probably designed to work great with unobtrusive asp.net validation but it may not be the best choice for a custom SPA implemented in AngularJS. Notice, that it would be hard to iterate over all properties that contain validation errors.

{% highlight C# %}
public class RegistrationController : ApiController
{
    public Registration Post(Registration registration)
    {
        if (!ModelState.IsValid)
        {
            HttpResponseMessage errMsg = Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
            throw new HttpResponseException(errMsg);
        }

        return registration;
    }
}
{% endhighlight %}

{% highlight javascript %}
{
    "Message":"The request is invalid.",
    "ModelState":{
        "registration.CarManufacturer.Id":[
            "The field Id must match the regular expression '^(BU|FD|FT|WV)$'."
        ]
    }
}
{% endhighlight %}

Fortunately, it is easy to design your own json format by just using `CreateResponse` call instead of `CreateErrorResponse`. The simplest, naive example would be to return flat list of validation errors:

{% highlight C# %}
public Registration Post(Registration registration)
{
    if (!ModelState.IsValid)
    {
        var validationErrorMessages = new List<string>();
        foreach (var modelStateKey in ModelState.Keys)
        {
            validationErrorMessages.AddRange(ModelState[modelStateKey].Errors.Select(error => error.ErrorMessage));
        }

        HttpResponseMessage msg = Request.CreateResponse(HttpStatusCode.BadRequest, validationErrorMessages);
        throw new HttpResponseException(msg);
    }

    return registration;
}
{% endhighlight %}
