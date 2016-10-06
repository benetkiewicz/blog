---
layout: post
title: "Code navigation in loosely coupled project architecture"
date: "2016-10-06 20:26"
categories: mvc rest webapi
---

When your web application project has App and Web layer, it is often not only divided conceptually but also on a physical and infrastructure layer. No matter if communication between layers is via WCF, asmx or .NET remoting (anyone here remembers .NET remoting?), you have a nice and clean separation with ability to scale but also a bit of a pain in code navigation. Drilling down using _Go To Implementation_ finally get you to WCF proxy class, HTTP call or something similarly useless. Then you need to understand how the technology works and find "the other end".
Fortunately there are (simple) ways to tackle this problem. I really like [this guy's approach](http://www.codemag.com/article/0809101) on WCF. I use it in some of my projects and it works really well.

Similar trick works for REST. If you are in control of both sides - REST service and a client - you can save yourself from code navigation pain with a simple interface. It is no rocket science but so many projects don't follow this pattern and precious minutes of your life are wasted for unnecessary _ctrl+F_.

Imagine having the following code on a REST service side:

```csharp
[RoutePrefix("api/v1/user")]
public class UserController : ApiController, IUserService
{
    [Route("{userId}")]
    public UserDto GetUser(int userId)
    {
        UserDto user = this.userRepo.GetById(userId);
        return user;
    }
}
```

and the following in the client:

```csharp
public UserDto GetUser(int userId)
{
    var client = new HttpClient();
    client.BaseAddress = new Uri("http://localhost:2460/");
    var response = client.GetAsync("api/v1/user/" + userId).Result;
    if (!response.IsSuccessStatusCode)
    {
        return null;
    }

    string responseJson = response.Content.ReadAsStringAsync().Result;
    UserDto result = JsonConvert.DeserializeObject<UserDto>(responseJson);
    return result;
}
```

See the pattern?

```csharp
public interface IUserService
{
    UserDto GetUser(int userId);
}
```

So now, every time you use your interface like below, you can _Go To Implementation_ and easily see what's going on on both sides of the wire.

```csharp
IUserService user = new UserServiceClient();
UserDto userDto = user.GetUser(userId);
```

### Note about RESTfulness

Returning `null` from `GetUser` method on server side is not very RESTful, yet we cannot return `IHttpActionResult` because we need to implement interface. Fortunately there's a way:

```csharp
[RoutePrefix("api/v1/user")]
public class UserController : ApiController, IUserService
{
    [Route("{userId}")]
    public UserDto GetUser(int userId)
    {
        UserDto user = this.userRepo.GetById(userId);
        if (userDto == null)
        {
            throw new HttpResponseException(System.Net.HttpStatusCode.NotFound);
        }

        return user;
    }
}
```

This doesn't address the potential problem with RESTful __PUT__ action (`CreateUser`) but creative programmer with figure something out using `ActionFilterAttribute`.

See the whole sample [on my github](https://github.com/benetkiewicz/RestApiCodeNavigationSample).
