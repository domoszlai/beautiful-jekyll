---
layout: post
title: Monadic templating in C#
subtitle: Part 1 
excerpt: In this article I propose a framework for generating formatted output using distributed generator functions. The framework is based on a technique heavily exploited in the functional programming world, called monads. 
description: In this article I propose a framework for generating formatted output using distributed generator functions. The framework is based on a technique heavily exploited in the functional programming world, called monads. 
tags: [functional programming, C#]
show-avatar: true
gh-repo: domoszlai/MonadicTemplate
gh-badge: [star, fork, follow] 
---

{% include note.html content="This article was originally published on [TallComponents.com](http://TallComponents.com)" %}

## Introduction

When one is to generate formatted output from a complex hierarchical set of data, in our example, generating source code from an abstract syntax tree (AST), the task must be or practical to be divided into smaller subtasks, small generator functions which are responsible for producing output of a specific part of the data.

The output of the generator functions must be collected, then concatenated or combined together in a complex way using some templating technique. Speaking about string outputs, concatenation also must be done in an efficient manner using helper classes e.g. [StringBuilder](https://msdn.microsoft.com/en-us/library/system.text.stringbuilder(v=vs.110).aspx). In addition, the individual generator functions probably share some generation context with non-trivial scoping. They may generate warnings or errors along with the formatted output, which also must be collected. And the requirements can go on...

All of these requirements affect the calling convention between the individual generator functions, and it can be very complex even in a single case. The generation context must be passed, multiple return values must be received: generated output, warnings, modified context... It is also very hard to change the calling convention later as it easily pervades the whole source code.

In this post we explain a framework for generating such formatted output using distributed generator functions. The framework is based on a technique heavily exploited in the functional programming world, called [monads](https://wiki.haskell.org/Monad). With this technique arbitrary complex calling convention can be hidden in a small library, which can be extended later without affecting the actual generation logic.

In this first part of the post, the monadic generator functions are explained in details. The generator functions will be very lightweight in usage while can efficiently collect and concatenate the output of the generator functions substasks are delegated to. In our explanation example, an AST based source code generator, the generator functions share a generator context. The context is immutable and/thus the scope of the changes can be freely managed.

In the second part of the tutorial, the framework will be extended to provide lightweight monadic templating using the [string interpolation](https://msdn.microsoft.com/en-us/library/dn961160.aspx) feature of C# 6.0.

Throughout the tutorial, we stick with the immutable approach as we believe that it results in less error prone software in complex tasks. As Eric Lippert phrases in his blog [post](http://blogs.msdn.com/b/ericlippert/archive/2007/10/04/path-finding-using-a-in-c-3-0-part-two.aspx):

> ASIDE: **Immutable data structures are the way of the future in C#**. It is much easier to reason about a data structure if you know that it will never change. Since they cannot be modified, they are automatically threadsafe. Since they cannot be modified, you can maintain a stack of past “snapshots” of the structure, and suddenly undo-redo implementations become trivial. On the down side, they do tend to chew up memory, but hey, that’s what garbage collection was invented for, so don’t sweat it.

The source code of the example used throughout this post can be downloaded from the following GitHub repository: [https://github.com/domoszlai/MonadicTemplate](https://github.com/domoszlai/MonadicTemplate). The example is based on C# 6.0 features as string interpolation and static imports, thus one needs VisualStudio 2015 to be able to compile it.

## Generator functions

In the following, let us suppose that we have the task already mentioned in the introduction, namely, generating source code given a hierarchical AST. We will use this example throughout the post.

First, let us see how a generator function would look developed in a naive way. We are basically doing string concatenations, so it must be done carefully to be efficient enough. Using `StringBuilder` is a trivial choice, the first approach could be to pass around a `StringBuilder` instance. Unfortunately this approach is not compositional, the result is messy, hard to read and maintain; templating cannot be easily achieved either, because of the lack of compositionality:

```csharp
void GenerateMethod(StringBuilder o, Method method) 
{ 
   o.Append("function "); 
   GenerateIdentifier(o, method.Name); 
   o.Append("("); 
   GenerateArgumentList(o, method.Arguments); 
   o.Append("){"); 
   GenerateBlock(o, method.Body); 
   o.Append("}"); 
}
```

Having a string return type results in more readable code and it comes only with a small price of managing a local StringBuilder. Templating is also possible with this approach:

```csharp
{% raw %}string GenerateMethods(List<Method> methods)
{ 
   var o = new StringBuilder();
 
   foreach(var method in methods) 
   { 
      var name = GenerateIdentifier(method.Name); 
      var arguments = GenerateArgumentList(method.Arguments); 
      var body = GenerateBlock(method.Body);
      o.AppendFormat("function {0}({1}){{{2}}}", name, arguments, body);
   }
  
   return o.ToString();
}{% endraw %}
```

It is getting more complicated though if generation context is introduced. To keep the solution compositional, the context is passed around in a "ref" parameter (another design could be to return a tuple of values from the generator functions as it would be done in a functional language):

```csharp
{% raw %}string GenerateMethods(ref Context ctx, List<Method> methods) 
{ 
   var o = new StringBuilder();
   
   foreach(var method in methods) 
   { 
      var name = GenerateIdentifier(ref ctx, method.Name); 
      var arguments = GenerateArgumentList(ref ctx, method.Arguments); 
      var body = GenerateBlock(ref ctx, method.Body);
      o.Append($"function {name}({arguments}){{{body}}}");
   }

   return o.ToString(); 
}{% endraw %}
```

Implementation details are getting pervades the source code now. If more features are added, e.g. generated warnings, writing generator functions are getting very tiresome and error prone. This is the point where functional programming "design patterns" come to the rescue.

## Monadic generators

The technique we will use, called monad,was introduced by [Phillip Wadler](http://homepages.inf.ed.ac.uk/wadler/) in his seminal paper [Monads for functional programming](http://homepages.inf.ed.ac.uk/wadler/papers/marktoberdorf/baastad.pdf) in 1992.

According to [Wikipedia](https://en.wikipedia.org/wiki/Monad): 

> In functional programming, a monad is a structure that represents computations defined as sequences of steps: a type with a monad structure defines what it means to chain operations, or nest functions of that type together.

In other words, a monad is a data structure which helps to define steps of computations in a compositional way: basic computation steps can be composed together to an indistinguishable new step. A monad usually has two operations associated with it:

1. *return* lifts a constant value into a monad;
2. *bind* or >>= composes to computations into a new computation in a sequential manner. a >>= b means that a computation precedes b computation.

In our case the monad type will be the `Generator<T>` class. The is a bit more general than it was in the introductory example, but we want to enable the generator functions to be able to produce any value, not just strings. For practical reasons the `T` type variable must be covariant, thus an interface also must be utilized. It contains only one function to execute the monad (we will discuss this concept later):

```csharp
interface IGenerator<out T> 
{ 
   ICovariantTuple<T, Context> Run(Context ctx); 
}
```

The actual `Generator<T>` class simply wraps a generator function, a delegate called `GFun<T>`. The wrapping can be done using the `Wrap` function. An arbitrary value can be lifted into the monadic context using the `Return` function. The bind operation is basically the `+` operator. This latter is a bit limited as it always returns `Generator<string>` this is enough for our purposes, the generic nature of the `Generator<T>` class will be exploited in the templating code.

```csharp
public delegate ICovariantTuple<T, Context> GFun<T>(Context ctx);
 
public class Generator<T> : IGenerator<T>
{
  private readonly GFun<T> f;

  private Generator(GFun<T> f)
  {
     this.f = f;
  }

  private static Generator<T> Wrap<T>(GFun<T> f)
  {
     return new Generator<T>(f);
  }

  public ICovariantTuple<T, Context> Run(Context ctx)
  {
     return f(ctx);
  }

  public static Generator<T> Return(T val)
  {
     return Generator<T>.Wrap(delegate (Context ctx)
     {
        return CovariantTuple<T, Context>.Create(val, ctx);
     });
  }

  public static Generator<string> operator +(Generator<T> a, Generator<T> b)
  {
     return Wrap(delegate (Context ctx)
     {
        StringBuilder o = new StringBuilder();

        var Res1 = a.f(ctx);
        o.Append(Res1.Item1);

        var Res2 = b.f(Res1.Item2);
        o.Append(Res2.Item1);

        return CovariantTuple<String, Context>.Create(o.ToString(), Res2.Item2);
     });
  }
} 
```

### Basic examples

And this is how to use our monad (in the following we skip the class name from method invocations, e.g. `Return("string")` instead of `Generator<string>.Return("string")` this can be done anyway using *static* imports). The simplest possible example is lifting a constant to the `Generator` monad:

```csharp
Generator<string> res = Return("str");
```

Let us contemplate a bit on this. `Return("str")` has the type of `Generator<string>` which is just a wrapped delegate having an argument of type `Context`. That is, when a `Generator` class is created, only a closure is created, a kind of suspended computation which is not executed until the context argument is finally provided. This is where the `Run` method of the `IGenerator<T>` interface comes into the picture:

```csharp
string str = res.Run(new Context()).Item1; // Item2 contains the modified context
```

Then again, when two generators are composed using the bind operation `(+)` the result is also a suspended computation, none of the parts are executed until the generation context is finally applied to the compound generator:

```csharp
Generator<string> res = Return("str1") + Return("str2");
```

Running res results in a cascade of recursive executions of the embedded generators. The delegate related to the `(+)` operator takes the first generator, executes it using the provided context then executes the second computation using the context returned by the first computation (the values are concatenated using an internal `StringBuilder`).

```csharp
string str = res.Run(new Context()).Item1; // == "str1str2"
```

### Accessing to the context

For a more complex example we need to introduce two more methods for the `Generator<T>` class. These methods enable to access to the current context:

```csharp
public delegate Generator<T> GContextFun<T>(Context ctx);

public class Generator<T> : IGenerator<T> 
{ 
   ...

   public static Generator<T> WithContext(GContextFun<T> gfun) 
   { 
      return Generator<T>.Wrap(delegate (Context ctx) 
      { 
         return gfun(ctx).Run(ctx); 
      }); 
   }

   public static Generator<T> SetContext(Generator<T> gen, Context localctx) 
   { 
      return Generator<T>.Wrap(delegate (Context ctx) 
      { 
         var ret = gen.Run(localctx); 
         return CovariantTuple<T, Context>.Create(ret.Item1, ctx); 
      }); 
   }
}
```

The `WithContext` method wraps a `GContextFun<T>` into a generator `GContextFun<T>` enable read-only access to the current generation context. The `SetContext` method enables to change the context with a local scope (for a given generator; the context returned by the generator is dropped in line 20). The following utility functions demonstrate their usage. It is supposed that the `Context` class contains the pair of `GetProperty`/`SetProperty` methods:

```csharp
public Generator<string> GetContextProperty(string property) 
{ 
   return WithContext(delegate (Context ctx) 
   { 
      return ctx.GetProperty(property); 
   }); 
}

public Generator<T> SetContextProperty<T>(Generator<T> gen, string property, Generator<string> value)
{ 
   return WithContext(delegate (Context ctx) 
   { 
      return SetContext(gen, ctx.SetProperty(property, value)); 
   }); 
}
```

### A more realistic example

Now we can have a quick glimpse into how real code, based on the `Generator<T>` class, would look like. The `Template` method used in the following example is still a debt of us. It is already uses a monadic extension of the string interpolation feature of C# 6.0, this will be introduced in the second part of this blog post series. Still, you can see how much easier to read it than the `String.Format` based version used in the introductory examples:

```csharp
{% raw %}public Generator<string> GenerateBlock(Block block) 
{ 
   return R("Method body for ") + GetContextProperty("method"); 
}

public Generator<string> GenerateMethod(Method method) 
{ 
   var name = GenerateName(method); 
   var arguments = GenerateArguments(method); 
   var body = SetContextProperty(GenerateBody(method), "method", name);
   return Template($"function {name}({arguments}){{{body}}}"); 
}{% endraw %}
```

Much nicer, isn't it?

## Evaluation

As the previous example shows, using the introduced framework, we can concentrate on the pure "business" logic, the code is very readable and easy to extend. It is also efficient enough; F# also runs on CLR, it is prepared to deal with higher order functions efficiently. These are pros, but what are the cons?

First of all, it increases the conceptual complexity of the code a great deal. Implementation details are now hidden behind higher order functions; that can make the inexperienced hard to see what happens under the hood. Further disadvantage is that with this monadic approach we introduced a kind of [lazyness](https://en.wikipedia.org/wiki/Lazy_evaluation). When a generator function is executed it just "prescribes" the computation steps, the actual execution is postponed until the monad is executed. This can make debugging a challenge in some cases.

In [Part 2](http://dlacko.org/blog/2015/11/16/monadic-templating-in-csharp-part-2/), I show how to coupage templating with the string interpolation feature of C# 6.

