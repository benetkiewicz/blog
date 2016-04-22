---
layout: post
title: "Transitive Group queries in Azure AD Graph API"
date: "2016-04-22 22:08"
categories: azure graph
---
Some group operations are transitive and others are scoped only to direct members of the group.

I need to get all groups that user is assigned to. I had the following code:

```csharp
IUserFetcher userFetcher = graphClient.Users.GetByObjectId(userObjectId);
IPagedCollection<IDirectoryObject> pagedCollection = await userFetcher.MemberOf.ExecuteAsync();
```

I was all nice until nested groups appeared. `MemberOf` is not transitive, so I needed something else. According to documentation, I can do something like:

```csharp
var groupIds = await userFetcher.GetMemberGroupsAsync(false);
```

but that gives me a collection of objectIds and not the rich object. How can I query graph for list of `Group` objects that I am interested in, not falling into n+1 trap using `graphClient.Groups.GetByObjectId()`? Note that `graphClient.Groups.Where()` clause is limited solely to displayName startsWith filter and every other interesting predicate will throw an exception.

I know about batches but still, some filtering would be an obvious choice.

Finally, after hours of experimenting and googling, I found `GetObjectsByObjectIdsAsync` method of `ActiveDirectoryClient` itself.
