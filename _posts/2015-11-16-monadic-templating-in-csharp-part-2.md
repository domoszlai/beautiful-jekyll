---
layout: post
title: Monadic templating in C#
subtitle: Part 2 
excerpt: In this article, first, string interpolation is introduced briefly by contrasting it with some earlier approaches. Following that I'll show how to marry string interpolation with the previously introduced monadic generators. 
description: In this article I propose a framework for generating formatted output using distributed generator functions. The framework is based on a technique heavily exploited in the functional programming world, called monads. 
tags: [functional programming, C#]
show-avatar: true
gh-repo: domoszlai/MonadicTemplate
gh-badge: [star, fork, follow] 
---

{% include note.html content="This article was originally published on [TallComponents.com](http://TallComponents.com)" %}

In the previous article ([Part 1](http://dlacko.org/blog/2015/11/16/monadic-templating-in-csharp-part-1/)), I had introduced a prototype of a monadic framework for generating formatted output (e.g. source code) from some complex, hierarchical set of data (e.g. Abstract Syntax Tree of some input language). This second part picks up where the first part left, the explanation of the actual templating by utilizing the string interpolation feature of C# 6.0.

## Introduction

Generating formatted output with lightweight templates has never been easier than with the string interpolation feature of C# 6.0. This feature renders some (used to be very useful) templating libraries, e.g. SmartFormat obsolete. String interpolation basically means that one can insert arbitrary expressions into a string by decorating it with curly brackets. We can finally forget String.Format and the numbered placeholders...

In this article, first, string interpolation is introduced briefly by contrasting it with some earlier approaches. Following that I'll show how to marry string interpolation with the previously introduced monadic generators.

## Lightweight string templating

In the past we had two options for lightweight templates. The most obvious choice was the ubiquitous String.Format. It gets a template in its first argument, and uses [varargs](https://msdn.microsoft.com/en-us/library/w5zay9db.aspx) for passing the objects referenced by the template (the exact same [idea](http://www.cplusplus.com/reference/cstdio/printf/) used in C since the seventies). The template language is simplistic, references are indexes of the arguments wrapped into curly brackets (if one wants to output curly bracket characters, those must be duplicated for escaping):

```csharp
{% raw %}String.Format("function {0}({1}){{{2}}}", name, arguments, body){% endraw %}
```

It is a very unpractical approach as it is very hard to read. It is mainly because there of the indirection in the template and that the indirection is achieved by numbered indexes. Numbered indexes are great for computers, but highly demotivating for humans.

A better approach is what used by e.g. [`SmartFormat`](https://github.com/scottrippey/SmartFormat.NET/wiki), named placeholders:

```csharp
{% raw %}Smart.Format("function {name}({arguments}){{{body}}}", function){% endraw %}
```

Unfortunately, this feature is limited. It works only if the template has exactly one object argument (the names in the template correspond to the members of the object then). It helps a bit that [anonymous types](https://msdn.microsoft.com/en-us/library/bb397696.aspx) can be involved:

```csharp
{% raw %}Smart.Format("function {name}({arguments}){{{body}}}", new {name, arguments, body}{% endraw %}
```

It is not bad any more, only those indirect references wouldn't be there...

This is when [string interpolation](https://msdn.microsoft.com/en-us/library/dn961160.aspx) comes into the picture:

```csharp
{% raw %}String res = $"function {name}({arguments}){{{body}}}");{% endraw %}
```

It is just great. It directly provides strings, and arbitrary expression can be embedded in the placeholders. It is exactly what we need.

## Customizing string interpolation

Previously I was not completely honest, string interpolation actually does not provide string *directly*, instead of an object of type [`FormattableString`](https://msdn.microsoft.com/en-us/library/system.formattablestring.aspx). This can be stringified through its [`ToString`](https://msdn.microsoft.com/en-us/library/dn906198.aspx) method by a custom [`IFormatProvider`](https://msdn.microsoft.com/en-us/library/system.iformatprovider.aspx).

When we want to execute our generators (a kind of state monad), we need to provide an initial state.

Thus all we have to do to use `Generator<String>` types expressions with string interpolation, is to use a stateful `IFormatProvider`:

```csharp
class GFormatProvider : IFormatProvider, ICustomFormatter 
{ 
   private Context ctx;
 
   public GFormatProvider(Context ctx)
   {
      this.ctx = ctx;
   }

   public object GetFormat(System.Type formatType)
   {
      if (formatType == typeof(ICustomFormatter)) return this;
      return null;
   }
   
   public string Format(string format, object arg, IFormatProvider formatProvider)
   {
      if (arg == null)
      return string.Empty;

      // This is why we need the covariant type variable
      if (arg is IGenerator<object>)
      {
         arg = ((IGenerator<object>)arg).Run(ctx).Item1;
      }

      if(arg is IFormattable)
      {
         return ((IFormattable)arg).ToString(format, formatProvider);
      }
      else
      {
         return arg.ToString();
      }
   }
}
```

The provider is instantiated with a `Context` (the modified `Context` is dropped on purpose, but it could be returned if required). The `Format` method is executed for every placeholder, and it speaks for itself. It's worthwhile to note though that this is where we exploit that `IGenerator<T>` is covariant. Otherwise, an arbitrary `IGenerator<T>` couldn't be cast to `IGenerator<object>`.

The only missing piece now is the utility method in `Generator<T>` to hide the custom `IFormatProvider`:

```csharp
public class Generator<T> : IGenerator<T> 
{ 
   ...

   public static Generator<String> Template(FormattableString formattable)
   {
      return Wrap(delegate (Context ctx)
      {
         var provider = new GFormatProvider(ctx);
         var res = formattable.ToString(provider);
         return CovariantTuple<String, Context>.Create(res, ctx);
      });
   }
}
```

## Conclusion

With this straightforward extension to string interpolation, our prototype monadic generator/templating library is ready to make experiences. I hope its complexity does not hide the beauty in this monadic approach.

And finally, as usual, the completely source code can be downloaded from the following GitHub repository [https://github.com/domoszlai/MonadicTemplate](https://github.com/domoszlai/MonadicTemplate)
