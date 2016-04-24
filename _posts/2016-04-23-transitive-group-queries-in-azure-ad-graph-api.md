---
layout: post
title: Transitive Group queries in Azure AD Graph API
date: '2016-04-23 18:37'
categories: azure graph
---

In Windows Azure Graph API, some operations on groups are transitive and others are scoped only to direct members of the group. So if you have a user that a member of _developers_ group, and groups _reporting_ and _admin_ contain _developers_, non-transitive operation will not tell you that your user is member of _reporting_.

I needed to get all group names that user is assigned to. I had the following code:

```csharp
IUserFetcher userFetcher = graphClient.Users.GetByObjectId(userObjectId);
IPagedCollection<IDirectoryObject> pagedCollection = await userFetcher.MemberOf.ExecuteAsync();
// further processing
```

I was all nice until nested groups appeared. `MemberOf` is not transitive, so I needed something else. According to documentation, I can do something like:

```csharp
var groupIds = await userFetcher.GetMemberGroupsAsync(false);
```

but that gave me a collection of _objectIds_ and not the rich object. I needed to query Graph API for list of Group objects that I am interested in, not falling into n+1 trap using `graphClient.Groups.GetByObjectId()`. Note that `graphClient.Groups.Where()` clause is limited solely to _displayName_ _startsWith_ filter and every other interesting predicate will throw an exception.

I know about batches but still, some filtering would be an obvious choice.

Finally, after hours of experimenting and googling, I found `GetObjectsByObjectIdsAsync` method of `ActiveDirectoryClient` itself.
Now the following code works:

```csharp
IUserFetcher userFetcher = graphClient.Users.GetByObjectId(userObjectId);
IEnumerable<string> userGroupIds = await userFetcher.GetMemberGroupsAsync(false);
List<string> userGroupIdList = userGroupIds.ToList();
IEnumerable<IDirectoryObject> userGroups = await graphClient.GetObjectsByObjectIdsAsync(userGroupIdList, new List<string> {"group"});
foreach (IDirectoryObject directoryObject in userGroups)
{
    var group = directoryObject as Group;
    if (group != null)
    {
        groupNames.Add(group.DisplayName);
    }
}
```

Hope it will help someone in a similar situation. This is a horrible developer experience and I am seriously considering contributing to Graph project since all pieces seem to exist, someone just need to put them in a correct place.
