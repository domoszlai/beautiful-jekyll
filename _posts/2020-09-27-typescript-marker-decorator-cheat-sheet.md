---
layout: post
title: TypeScript decorator cheat sheet
subtitle: with examples
tags: [typescript, decorators]
show-avatar: true
description: Practical guide on how to use marker decorators on TypeScript data structures.
excerpt: Practical guide on how to use marker decorators on TypeScript data structures.
image: /img/typescript_logo.png
gh-badge: [star, fork, follow] 
---

## Overview

I've been developing a prototype for a library that offers some type annotations and
does post processing of instances of annotated types, and I was suprised that, in spite of the ample
blog posts/articles that describes TypeScript decorators, how hard is to find some simple examples
that explains both the annotation **and the detection part**. So here you are!

{% include note.html content="This article only provides examples, for detailed explanation of how
decorators work please refer to the [TypeScript documentation](https://www.typescriptlang.org/docs/handbook/decorators.html)" %}

## Decorator examples

The examples use the [`reflect-metadata`](https://github.com/rbuckton/reflect-metadata) library to make the
examples more concice, so don't forget to import:

```typescript
import "reflect-metadata"
```

and the following declaration is also used in all of the examples:

```typescript
const markedKey = Symbol("marked")
```

### Class decorators

Consider the class annotation `marked_class`: 

```typescript
@marked_class
class A {
    ...
}
```

A simple decorator for the annotation:

```typescript
function marked_class<T>(
    constructor: T /* constructor of the class */
  ) {
    Reflect.defineMetadata(markedKey, "marked with default value", constructor)
}
```

or if you also want to pass parameters:

```typescript
@marked_class("my value")
class A {
    ...
}
```

a [decorator factory](https://www.typescriptlang.org/docs/handbook/decorators.html#decorator-factories) can be developed:

```typescript
function marked_class<T>(value: string){
    return function (
        constructor: T /* constructor of the class */
    ) {
        Reflect.defineMetadata(markedKey, value, constructor)
    }
}
```

And this is how to check if an object is instance of an annotated class:

```typescript
function getClassMark(obj: any) {
    return Reflect.getMetadata(markedKey, obj.constructor)
}
```

### Method decorators

Consider the method annotation `marked_method`: 

```typescript
class A {
    @marked_method
    doA(){...}
}
```

A simple decorator for the annotation:

```typescript
function marked_method<T>(
    target: T, /* prototype of the class or constructor function of the class for a static member */
    propertyKey: string,
    descriptor: PropertyDescriptor
) {
    Reflect.defineMetadata(markedkey, "marked with default value", target, propertyKey)
}
```

if you also want to pass parameters:

```typescript
class A {
    @marked_method("my value")
    doA(){...}
}
```

the [decorator factory](https://www.typescriptlang.org/docs/handbook/decorators.html#decorator-factories) can be:

```typescript
function marked_method<T>(value: string){
    return function (
      target: T, /* prototype of the class or constructor function of the class for a static member */
      propertyKey: string,
      descriptor: PropertyDescriptor
    ) {
        Reflect.defineMetadata(markedkey, value, target, propertyKey)
    }
}
```

And this is how to check if a method of an object is annotated:

```typescript
function getMethodMark(obj: any, methodName: string) {
    return Reflect.getMetadata(markedKey, obj, methodName)
}
```

### Property decorators

Consider the property annotation `marked_property`: 

```typescript
class A {
    @marked_property
    propertyA = ...
}
```

A simple decorator for the annotation:

```typescript
function marked_method<T>(
    target: T, /* prototype of the class or constructor function of the class for a static member */
    propertyKey: string
) {
    Reflect.defineMetadata(markedkey, "marked with default value", target, propertyKey)
}
```

{% include note.html content="Please note the missing `descriptor: PropertyDescriptor` parameter compared to method annotations" %}

if you also want to pass parameters:

```typescript
class A {
    @marked_property("my value")
    propertyA = ...
}
```

the [decorator factory](https://www.typescriptlang.org/docs/handbook/decorators.html#decorator-factories) can be:

```typescript
function marked_method<T>(value: string){
    return function (
        target: T, /* prototype of the class or constructor function of the class for a static member */
        propertyKey: string
    ) {
        Reflect.defineMetadata(markedkey, value, target, propertyKey)
    }
}
```

And this is how to check if a property of an object is annotated:

```typescript
function getPropertyMark(obj: any, propertyName: string) {
    return Reflect.getMetadata(markedKey, obj, propertyName)
}
```
