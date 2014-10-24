---
layout: post
title:  "Code mutation with Roslyn"
date:   2014-10-24 14:57:59
categories: csharp roslyn
---

On the [devday] conference I attended in September, there was this guy, [Seb Rose], who asked an interesting question:

`Is coverage a good metric?`

When you think about it, it may not be the best. Assuming you have the following code to test:

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

The following unit test will give you a pretty good coverage:

{% highlight C# %}
Debug.Assert(Calculator.MultiplyByTwo(0) != null);
{% endhighlight %}

But is that good? Obviously no. What if you could create code mutation and run the test again? Code mutation would be almost the same as before but with a subtle change, which would slightly change how the algorithm works and should be detected by unit test.
With that magic engine in place, we could define our new metric as percentage of failing tests, which are being run against all kinds of code mutations. Ideally, the best result is when only the original code works and the simplest semantical change breaks the test.

This idea became my __idée fixe__ for the last month and it encouraged my to make a reasearch on a scary topic, which is .NET compiler platform (__[Roslyn]__). Roslyn is advertised as Compiler as a Service which in my mind translates to API that allows you to work with a source code on both syntax and semantic level.

My take on code mutation is the simplest possible: find declared const integers and increment them by one. Run the tests again to see if they fail.

Here's my first test, no mutation applied:

{% highlight C# %}
[Test]
public void OriginalCalculatorCodeShouldWork()
{
    string calcCode = File.OpenText("Calculator.cs").ReadToEnd();
    SyntaxTree syntaxTree = SyntaxFactory.ParseSyntaxTree(calcCode);

    var initialCompilation = CreateCompilationForSyntaxTree(syntaxTree);
    Assembly initialAssembly = CreateAssemblyForCompilation(initialCompilation);
    RunCalcTestOnAssembly(initialAssembly);
}
{% endhighlight %}

Note it contains `CreateCompilationForSyntaxTree`, `CreateAssemblyForCompilation` and `RunCalcTestOnAssembly`, which are my helper methods for some Roslyn and Reflection magic to happen.

Code which in-memory compiles the source looks like that:

{% highlight C# %}
private CSharpCompilation CreateCompilationForSyntaxTree(SyntaxTree syntaxTree)
{
    return CSharpCompilation.Create(
        "calculator.dll",
        options: new CSharpCompilationOptions(OutputKind.DynamicallyLinkedLibrary),
        syntaxTrees: new[] { syntaxTree },
        references: new[] { new MetadataFileReference(typeof(object).Assembly.Location) });
}
{% endhighlight %}

Code which builds in-memory assembly is as follows:

{% highlight C# %}
private Assembly CreateAssemblyForCompilation(CSharpCompilation compilation)
{
    Assembly compiledAssembly;
    using (var stream = new MemoryStream())
    {
        compilation.Emit(stream);
        compiledAssembly = Assembly.Load(stream.GetBuffer());
    }

    return compiledAssembly;
}
{% endhighlight %}

And finally, some reflection to run the method from the in-memory assembly:

{% highlight C# %}
private void RunCalcTestOnAssembly(Assembly assembly)
{
    Type calculator = assembly.GetType("RoslynMutationTests.Calculator");
    MethodInfo evaluate = calculator.GetMethod("MultiplyByTwo");
    object result = evaluate.Invoke(null, new[] { (object)3 });

    Assert.AreEqual(6, result, string.Format("Expected 6, the result was {0}", result));
}
{% endhighlight %}

All of above is exactly the same as the following short test. It just builds the infrastructure for the Roslyn mutation magic:

{% highlight C# %}
[Test]
public void OriginalCodeSimpleTest()
{
    int result = Calculator.MultiplyByTwo(3);
    Assert.AreEqual(6, result);
}
{% endhighlight %}

The key tool for accomplishing my task is an abstract class `CSharpSyntaxRewriter`, which implements visitor pattern and allows to visit a variety of syntax tree types. In my case, I'm interested in `VisitLocalDeclarationStatement` method. When I'm in declaration node, I need to perform some tests first, to make sure if that piece of code is what I'm interested in:

* is this a constant declaration?
* is this `int` declaration
* is the variable initialized?

If any of the questions above is answered __no__, I'm just returning the node intact. If the given node is what I'm looking for, I read the const value, increment by one and build new node initialized with incremented value.

{% highlight C# %}
class IntIncrementingRewriter : CSharpSyntaxRewriter
{
    public SemanticModel SemanticModel { get; private set; }

    public IntIncrementingRewriter(SemanticModel semanticModel)
    {
        SemanticModel = semanticModel;
    }

    public override SyntaxNode VisitLocalDeclarationStatement(LocalDeclarationStatementSyntax node)
    {
        if (!node.IsConst)
        {
            return node;
        }

        VariableDeclaratorSyntax currentConst = node.Declaration.Variables[0];
        if (!IsIntegerType(currentConst))
        {
            return node;
        }

        if (currentConst.Initializer == null)
        {
            return node;
        }

        string declaredName = currentConst.Identifier.ValueText;
        int currentConstVal = int.Parse(currentConst.Initializer.Value.ToString());
        int newConstVal = currentConstVal + 1;

        // http://roslynquoter.azurewebsites.net/
        return SyntaxFactory.LocalDeclarationStatement(
            SyntaxFactory.VariableDeclaration(SyntaxFactory.PredefinedType(
                SyntaxFactory.Token(SyntaxKind.IntKeyword).WithTrailingTrivia(SyntaxFactory.Space))
                )
                .WithVariables(
                    SyntaxFactory.SingletonSeparatedList(
                        SyntaxFactory.VariableDeclarator(SyntaxFactory.Identifier(declaredName))
                            .WithInitializer(SyntaxFactory.EqualsValueClause(
                                SyntaxFactory.LiteralExpression(
                                    SyntaxKind.NumericLiteralExpression,
                                    SyntaxFactory.Literal(SyntaxFactory.TriviaList(), newConstVal.ToString(), newConstVal, SyntaxFactory.TriviaList())
                                    )))
                        )))
            .WithModifiers(
                SyntaxFactory.TokenList(SyntaxFactory.Token(SyntaxKind.ConstKeyword).WithTrailingTrivia(SyntaxFactory.Space))
            );
    }

    private bool IsIntegerType(VariableDeclaratorSyntax variable)
    {
        var initialiserInfo = SemanticModel.GetTypeInfo(variable.Initializer.Value);
        var declaredType = initialiserInfo.Type;
        return declaredType.ToString() == "int";
    }
}
{% endhighlight %}

Don't worry too much about that return statement. It just how you build `const int two = 3;` statement in Roslyn syntax API. There's this awesome [Roslynquoter] page, which builds it for you.

Finally the failing test:

{% highlight C# %}
[Test]
public void MutatedCalculatorCodeShouldWork()
{
    string calcCode = File.OpenText("Calculator.cs").ReadToEnd();
    SyntaxTree syntaxTree = SyntaxFactory.ParseSyntaxTree(calcCode);
    var syntaxTreeRoot = (CompilationUnitSyntax)syntaxTree.GetRoot();

    var initialCompilation = CreateCompilationForSyntaxTree(syntaxTree);
    SemanticModel semanticModel = initialCompilation.GetSemanticModel(syntaxTree);
    var calcCodeRewriter = new IntIncrementingRewriter(semanticModel);
    var mutatedSyntaxTreeRoot = calcCodeRewriter.Visit(syntaxTreeRoot);

    var mutatedCompilation = CreateCompilationForSyntaxTree(mutatedSyntaxTreeRoot.SyntaxTree);
    Assembly mutatedAssembly = CreateAssemblyForCompilation(mutatedCompilation);
    RunCalcTestOnAssembly(mutatedAssembly);
}
{% endhighlight %}

As you can see here, transforming this into commercial product should be easy. Just implement a ton of other mutations, provide some automation, reporting and easy to use API ;) But seriously, Roslyn is a great tool. I just can't wait for .net vNext, which will have Roslyn as one of its building blocks. I also can't wait for all the great tools that inevitably will follow, because the potential here is even hard to imagine.

[devday]: http://devday.pl
[Seb Rose]: https://twitter.com/sebrose
[Roslyn]: http://msdn.microsoft.com/en-us/vstudio/roslyn.aspx
[Roslynquoter]: http://roslynquoter.azurewebsites.net/
