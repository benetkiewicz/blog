---
layout: post
title: Aggregate string concatenation in TSQL explained
date: '2016-10-12 22:09'
categories: sql tsql
---

I'm not sure about latest version of Microsoft SQL Server, but still popular 2008R2 and 2012 versions do not have an aggregate function that would join strings for you. You need to choose one of the work-arounds:
1. a lengthy query that is using PIVOT
2. a CLR aggregator function
3. FOR XML
4. Common Table Expression (CTE)

Recently I use a lot of XML-type columns and queries, so I choose XML-based solution. Let's take baby steps.

Having this:

```sql
SELECT id, val FROM #tbl
```

 &nbsp;| id| val
--|---|--
 1| 1 | aaa
 2| 2 | bbb
 3| 1 | ccc

We want to get the following result:

&nbsp;| id| foo
--|---|--
 1| 1 | aaa, ccc
 2| 2 | bbb

Do you know that you can get a query result in XML by using `FOR XML PATH` directive.

```sql
SELECT * FROM #tbl FOR XML PATH
```

Result is:

```xml
<row>
  <id>1</id>
  <val>aaa</val>
</row>
<row>
  <id>1</id>
  <val>ccc</val>
</row>
<row>
  <id>2</id>
  <val>bbb</val>
</row>
```

`PATH` directive can be tweaked, to adjust the shape of resulting XML. You can tell how `<row>` node should be named.

```sql
SELECT * FROM #tbl FOR XML PATH('awesome')
```

```xml
<awesome>
  <id>1</id>
  <val>aaa</val>
</awesome>
<awesome>
  <id>1</id>
  <val>ccc</val>
</awesome>
<awesome>
  <id>2</id>
  <val>bbb</val>
</awesome>
```

So what? Let's get to the point. The point is you can also make the node name empty, which is where things become interesting:

```sql
SELECT id, val FROM #tbl FOR XML PATH('')
```

```xml
<id>1</id>
<val>aaa</val>
<id>1</id>
<val>ccc</val>
<id>2</id>
<val>bbb</val>
```

You know what happens when you mess around with your columns, like adding an empty string to the result? SQL Server will not generate column name for you. You'll get _(No column name)_ in result header. I wonder how this will be represented in XML result...

```sql
SELECT id+'', val+'' FROM #tbl FOR XML PATH('')
```

```
1aaa1ccc2bbb
```

That's the core of this hack. Now it is a matter of proper grouping, joining and cleanup.

```sql
SELECT id, (SELECT val+', ' FROM #tbl tblinner WHERE (tblinner.id=tblouter.id) FOR XML PATH('')) AS foo
FROM #tbl tblouter GROUP BY id
```

&nbsp;| id| foo
--|---|--
 1| 1 | aaa, ccc,
 2| 2 | bbb,


Now use `replace()` and `len()` or `stuff()` and voilla!
