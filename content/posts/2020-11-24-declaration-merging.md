---
title: "TypeScript: Declaration merging as an extension"
date: 2020-11-24T18:43:00+0800
tags: [note, typescript, extension, declaration merging, factory pattern]
categories: technique
---

If you have written Swift, there is an [extension pattern](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html) that is pretty neat and handy.
You can define customized behaviors on top of some existing class, even primitives, like this:

```swift
// borrow from swift documentation.
extension Double {
    var km: Double { return self * 1_000.0 }
    var m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
```

When I was first writing in typescript, I was curious about if there is any way that can achieve this extension pattern...?

Fortunately, there does exist!

In typescript, there is a concept called [Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html). To put it simpler, declaration merging is that compiler merge two or more declaration into a single one.

#### A few examples

You can define interfaces with same name in different block of code, and declare variable of it with both attributes:

```typescript
interface A {
  a: number;
  b: number;
}

interface A {
  c: number;
}

let a: A = { a: 1, b: 2, c: 3 };
```

Merging namespaces:

```typescript
namespace A {
  let a: number;
}

namespace A {
  let b: number;
}

// which is equivalent to:
namespace A {
  let a: number;
  let b: number;
}
```

Merging namespace with enum, class, and interface is also possible.

Class + Namespace:

```typescript
class A {
  a: number = 123;
}

namespace A {
  export let b: number = 456;
}

const instance = new A();
console.log(instance.a); // 123

console.log(A.b); // 456
```

Enum + Namespace:

```typescript
enum A {
  B,
  C,
  D,
}

namespace A {
  export let E: number = 456;
}

A.B; // 0
A.C; // 1
A.D; // 2
A.E; // 456
```

### A little deep dive

It looks like magic in the way that a class can be mixed with namespace.
But under the hood what the compiler does is pretty straight forward.

Let's spend some time to look into the design of typescript.

An `identifier` in typescript may be one of three

- Namespace
- Type
- Value

According to the [Basic Concept](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#basic-concepts) section, there is a table that records what will be created when you make a declartion:

| Declaration Type | Namespace | Type | Value |
| ---------------- | --------- | ---- | ----- |
| Namespace        | V         |      | V     |
| Class            |           | V    | V     |
| Enum             |           | V    | V     |
| Interface        |           | V    |       |
| Type Alias       |           | V    |       |
| Function         |           |      | V     |
| Variable         |           |      | V     |

For example, when a class `Foo` is declared, both a type `Foo` and a value `Foo` are created.

- type: you can define a variable with type `Foo`.
- value: you create a constructor named `Foo()`.

#### Rule of thumb

As long as the compiler can tell what an identifier really means in the context, everything is alright.

Continue the example of merging class ans namespace:

```typescript
class A {
  a: number = 123;
}
// type: A is created.
// value: constructor A() is created.

namespace A {
  export let b: number = 456;
}
// namespace: A is created.
// value: value A.b is created.
```

Now, though the class and namespace has the same identifier `A`, depends on the context, the meaning can be totally different, and that's why the compiler does not complaint.

```typescript
// A is a namespace created by the declaration of namespace A.
// get the variable b in namespace A: 456.
A.b;

// A is a value created by the declaration of namespace A.
// it contains values that defined in namespace A.
let namespaceA = A;
namespaceA.b; // 456

// A is a value created by the declaration of class A.
// it refers to the constructor function of type A.
let a = new A();

// A is a type created by the declaration of class A.
let b: A;
```

In the typescript deep dive documentation, the [adding using a namespace](https://www.typescriptlang.org/docs/handbook/declaration-files/deep-dive.html#adding-using-a-namespace) section, it menthions:

> This is legal as long as it does not create a conflict. A general rule of thumb is that values always conflict with other values of the same name unless they are declared as namespaces, types will conflict if they are declared with a type alias declaration (type s = string), and namespaces never conflict.

You have to make sure your `identifier` does not conflict, the compiler is smart enough to resolve the namespace / type / value in the context, or you will get `error TS1128: Declaration or statement expected.`.

Try to run this code in your environment such as in `ts-node`.

```typescript
class A {
  static b: number = 123;
}

namespace A {
  export let b: number = 567;
}
```

It complaints with `[eval].ts:2:10 - error TS2300: Duplicate identifier 'b'.`.
Since `A.b` can potentially either be `123` or `456`.

### Using declaration merging to achieve named constructor pattern

Here let's make some use with declaration merging.

Let's say I have a enum in typescript called `Level` meaning how good you are at coding:

```typescript
export enum Level {
  Newbie
  Intermediate
  Expert
}
```

Somehow, you have a logic that say:

- if your `workOfYear` is less than or equal to 1, you are a `Newbie`
- if your `workOfYear` of year is greater than 1 and less than or equal to 5, you are an `Intermediate`
- if your `workOfYear` of year is greater than 5, you are an `Expert`

Then you can write a named constructor like this:

```typescript
export enum Level {
  Newbie
  Intermediate
  Expert
}

export namespace Level {
  export function fromWorkOfYear(workOfYear: number): Level {
    if (workOfYear <= 1) return Level.Newbie;
    else if (workOfYear <= 5) return Level.Intermediate;
    else return Level.Expert;
  }
}
```

Now, you can simpily use `Level` like this:

```typescript
// normal usage.
const level1 = Level.Newbie;

// convert workOfYear to enum, effectly gives Level.Expert.
const level2 = Level.fromWorkOfYear(13);
```

Actually, it is just like a feature called [named constructor](https://dart.dev/guides/language/language-tour#constructors) in dart, which I found very useful.
