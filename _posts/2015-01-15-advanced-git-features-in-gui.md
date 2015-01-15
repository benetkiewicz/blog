---
layout: post
title:  "Advanced git features in GUI"
date:   2015-01-15 09:57:59
categories: git
---

There are three _advanced_ git commands that I use quite often to keep my commits (and entire history of the branch) clean and tidy. These are:

* partial add: `git add -p someFile.cs`
* interactive rebase on self: `git rebase -i HEAD~3`
* hard or mixed (O my Gosh, I forgot something mode) reset: `git reset --mixed HEAD^`

As you can see above I'm rather git-console person but I thought it might be fun to see how easy these things can be achieved in modern git GUIs for Windows. I chose three of them for my experiment:

* [GitHub for Windows][github] (my current favourite)
* [Git Gui][gitgui] (part of default msysgit installation)
* [SourceTree][sourcetree] (seems to be most popular GUI in the community)

<blockquote>
I use github for windows right now because it provides really pretty diff view (easier to use than going file after file when running `git diftool`) and I also use it for selecting files for stage/commit (easier to select a file with a mouse than type file names one by one).
</blockquote>

Scenario for my test is as follows - imagine you have the C# file that looks like this:

{% highlight C# %}
public class Calculator
{
    public static int MultiplyByTwo(int number)
    {
        const int two = 2;
        return number*two;
    }
}
{% endhighlight %}

and you apply two changes: add __sealed__ modifier to a class and remove __const__ modifier from _two_ variable.


{% highlight diff %}
--- a/calculator.cs
+++ b/calculator.cs
@@ -1,8 +1,8 @@
-public class Calculator
+public sealed class Calculator
{
    public static int MultiplyByTwo(int number)
    {
-        const int two = 2;
+        int two = 2;
        return number*two;
    }
}
{% endhighlight %}

Perfect GUI for Git should allow you to do three things:

* create two commits for these two changes respectively
* run interactive rebase, where you could squash one into another
* reset that resulting commit in mixed mode, so you can perform another change

### Github for Windows
At the time I am writing this, version 2.7.0.24 has just been released with partial add as a new feature.

![GitHub partial add]({{ site.url }}images/github_add_lines.png)

Unfortunately I don't see a way to interactively rebase or reset history to a given point.

### Default Git Gui
I was surprised that simple, ugly looking default Git Gui does have partial add functionality:

![GitGUI partial add]({{ site.url }}images/gitgui_add_lines.png)

I didn't find a way to perform an interactive rebase.

GitGUI allows reset operations:

![GitGUI reset]({{ site.url }}images/gitgui_reset.png)

It displays the following dialog window, where one can choose the type of reset:

![GitGUI reset dialog]({{ site.url }}images/gitgui_reset_dialog.png)

### SourceTree

This one is a real git power horse. It has all three tested features.

Partial add:

![SourceTree partial add]({{ site.url }}images/sourcetree_add_lines.png)

Interactive rebase:

![SourceTree interactive rebase]({{ site.url }}images/sourcetree_interactive.png)

![SourceTree interactive rebase dialog]({{ site.url }}images/sourcetree_interactive_dialog.png)

Reset:

![SourceTree reset]({{ site.url }}images/sourcetree_reset.png)

![SourceTree reset]({{ site.url }}images/sourcetree_reset_dialog.png)

## Verdict
SourceTree is a definite winner in this contest. It may not be as pretty as GitHub for Windows but I think I can get use to it. And when it comes to functionality, it is packed with advanced features and it looks like you can give your console day off if you have SourceTree installed.

[github]: https://windows.github.com/
[gitgui]: https://msysgit.github.io/
[sourcetree]: http://www.sourcetreeapp.com/
