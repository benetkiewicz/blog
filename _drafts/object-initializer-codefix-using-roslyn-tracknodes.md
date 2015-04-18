---
layout: post
title: "Object initializer code fix with Roslyn using TrackNodes extension method"
date: "2015-04-13 23:26"
---

### Code smell
One of the most annoying and non-obvious C# code smells is object initializer. I really hate them.

{% highlight C# %}
class Foo
{
    public Foo F { get; set; }
}

class Bar
{
    void Baz()
    {
        var f = new Foo() { F = null }; // Ouch!
    }
}
{% endhighlight %}

* They look ugly in the code, it is hard to format them in a proper way
* They are harder to debug (especially in VS 2010), when you put a breakpoint on a single initializer line, Visual Studio puts breakpoint on entire initializer
* They blur stack trace. If you have complex object initializer with nested properties being read (or God forbids - methods), your stack trace is still pointed to a first line of initialization statement.

These are the three main reasons which make me remove object initializers from code every time I can. Unfortunately this is a pain. Resharper seems to be fine with object initializer and refactoring by hand is clunky and not a lot of fun. Fortunately with Roslyn we can automate process of refactoring affected code.

### Analyzer
My solution will detect and fix only the simplest form of the object initializer construct used for local variables. It explicitly ignores:

* nested object initializers:

`Foo f = new Foo() { F = new Foo() { F = null } }`

* multiple variable declarations inside a single statement:

`Foo g = null, f = new Foo() { F = null };`

* object initializers used in parameters, property assignments, etc.:

`ProcessFoo(new Foo { F = null });`

With the above assumptions, analyzer code is clean and simple. I register for `LocalDeclarationStatement` type of nodes, check if there's only one object initializer per declaration and report diagnostic:

{% highlight C# %}
public override void Initialize(AnalysisContext context)
{
    context.RegisterSyntaxNodeAction(AnalyzeObjectInitializer, SyntaxKind.LocalDeclarationStatement);
}

private void AnalyzeObjectInitializer(SyntaxNodeAnalysisContext context)
{
    var localDeclarationExpression = context.Node as LocalDeclarationStatementSyntax;
    if (localDeclarationExpression == null)
    {
        return;
    }

    var innerObjectInitializers = localDeclarationExpression.DescendantNodes().OfType<InitializerExpressionSyntax>().ToList();
    if (innerObjectInitializers.Count != 1)
    {
        return;
    }

    var objectInitializer = innerObjectInitializers[0];
    context.ReportDiagnostic(Diagnostic.Create(Rule, objectInitializer.GetLocation()));
}
{% endhighlight %}

### Code fix difficulties to overcome
Code fix is a lot more complex then most examples that you can find in Roslyn tutorials and let me tell you why. Most code fixes operate in a single context and they don't need to go beyond a language construct being analyzed and fixed. For example, the [popular if braces analyzer][36f76dca] fixes everything inside a particular IfStatementSytax and does not have to go beyond that syntax node. It just takes expression and embeds it in a block if necessary. For me, dealing with ObjectInitializerSyntax is not enough. I need to remove it but I also need to add new statements right after variable initialization that correspond to property initialization that just have been removed. That means that I need to play with a code block containing given local variable initialization statement.

Applying few modifications on a immutable syntax tree has an interesting implication. After you apply your first modification to, let's say code block, all your references to nodes in the original syntax tree are lost (or I should rather say: inapplicable). Consider the following example:
![roslyn block modification](\images\roslyn-block-modification.png)

After you perform the insert operation on Block, you cannot simply perform replace Statement1 with Statement4 in Block':

`Block'.Replace(Statement1, Statement4)`

There is no Statement1 in Block' anymore! You must recalculate reference to Statement1' inside Block' somehow.

### TrackNodes() and GetCurrentNode() usage in code fix
Fortunately Roslyn covers the above scenario in a very convenient way. There are two extension methods supporting scenario I just described:

{% highlight C# %}
public static TRoot TrackNodes<TRoot>(this TRoot root, params SyntaxNode[] nodes)
    where TRoot : SyntaxNode
public static TNode GetCurrentNode<TNode>(this SyntaxNode root, TNode node)
    where TNode : SyntaxNode
{% endhighlight %}

Basically you should use `TrackNodes()` method to register nodes you want to survive the transformation. Roslyn will put some unique ID in the syntax tree, so it will be possible to easily retrieve new node reference by old node reference from the transformed tree using `GetCurrentNode()`. Let's see it in action:
{% highlight C# %}
var block = GetContainingBlock(declarationWithInitializer);
var newBlock = block.TrackNodes(declarationWithInitializer);
var refreshedObjectInitializer = newBlock.GetCurrentNode(declarationWithInitializer);
newBlock = newBlock.InsertNodesAfter(refreshedObjectInitializer, initializedProperties).WithAdditionalAnnotations(Formatter.Annotation);
refreshedObjectInitializer = newBlock.GetCurrentNode(declarationWithInitializer);
newBlock = newBlock.ReplaceNode(refreshedObjectInitializer, declarationWithoutInitializer);
{% endhighlight %}

As usual, the full source of analyzer, code fix and unit tests can be found [on my github][mygithub].

[36f76dca]: http://jeremybytes.blogspot.com/2014/12/new-video-building-diagnostic-analyzer.html
[mygithub]: https://github.com/benetkiewicz/TgdNetRoslynSamples/
