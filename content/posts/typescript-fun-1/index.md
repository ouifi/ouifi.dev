---
title: "TypeScript Fun"
date: "2022-09-15"
publishDate: "2022-09-15"
Toc: true
tags:
  - javascript
  - typescript
  - tech
keywords: 
  - typescript
description: ""
---

# TypeScript Fun

## Intro

Ah, TypeScript. The "superset" of JavaScript that adds strict compile-time type checking. It is almost a meme at this point people doing crazy things in TypeScript. This includes making a full on database just with the type checker[^1], and implementing TypeScript in TypeScript[^2].[^3] Now I don't reckon myself as clever as these folks, but I wanted to show what someone who uses a TypeScript on a daily basis might think of. 

## Typing Key Value Equality

### The Problem

In my current role, we make good use of TypeScript `enum`s. These are often used for defining system constants which are (1) not likely to change often, (2) when they are changed need to be changed globally, and (3) should be referenced by a name rather than a value. 

For one such feature, my fellow developers were debating the value of implementing a `const enum` (to force people to use the variable name) vs. a union of string values. 

#### Example1: Using an enum

```tsx
export enum SYSTEM_VALUES {
    value1 = "value1",
    value2 = "value2",
    value3 = "value3"
}

type ComponentProps = {
    systemValue: SYSTEM_VALUES
};

export const Component = ({ systemValue }: ComponentProps) => {
    // use the systemValue prop
    return <div>{systemValue}</div>;
};

// Then when you use the component...
export const Example1 = () => {
    return <Component systemValue={SYSTEM_VALUES.value1}/>
};

// But this creates an error
export const BadExample1 = () => {
    // Type '"value1"' is not assignable to type 'SYSTEM_VALUES'. Did you mean 'SYSTEM_VALUES.value1'?
    return <Component systemValue="value1" />
};
```

#### Example 2: Using a union of strings

```tsx
export type SYSTEM_VALUES = 
    | "value1"
    | "value2"
    | "value3"

type ComponentProps = {
    systemValue: SYSTEM_VALUES
};

export const Component = ({ systemValue }: ComponentProps) => {
    // use the systemValue prop
    return <div>{systemValue}</div>;
};

// Then when you use the component...
export const Example2 = () => {
    return <Component systemValue="value1"/>
};

// But you cant reference by the variable name
export const BadExample2 = () => {
    return <Component systemValue={SYSTEM_VALUES.value1} />;
};
```

#### Example 3: A smarter enum

One simple solution is to construct a type from a const object or enum. This is what we ended up doing because it has relatively low complexity with just a little trick.

```tsx
export enum SYSTEM_VALUES {
    value1 = "value1",
    value2 = "value2"
}
// or
export const SYSTEM_VALUES_ALT = {
    value1: "value1",
    value2: "value2"
} as const;

type SYSTEM_VALUE = keyof typeof SYSTEM_VALUES; // or SYSTEM_VALUES_ALT

export type ComponentProps = {
    systemValue: SYSTEM_VALUE;
};

export const Component = ({ systemValue }: ComponentProps) => {
    //use the systemValue prop
    return <div>{systemValue}</div>;
};

// Then you have the best of both worlds!
export const Example3 = () => {
    return <Component systemValue={SYSTEM_VALUES.value1} />;
};

export const Example3B = () => {
    return <Component systemValue="value1" />;
};
```

I wont go into the virtues of any of these approaches, and the real example was slightly more complex. But this did make me think: "Surely there is a way to have a strongly typed way of referencing a list of strings by value or by the variable name". 

### The Solution

I knew there was a way to turn an array of strings into a union type from [this 2019 article](https://steveholgado.com/typescript-types-from-arrays/). From this, I began the solution.

```tsx
const SYSTEM_VALUES = [
    "value1",
    "value2"
] as const;

type SYSTEM_VALUE = SYSTEM_VALUES[number];
```

Obviously the array is declared `const` at the start, so why is there the `as const` as the end? This tells TypeScript this is actually of type `ReadonlyArray`, so the elements wont ever change. Without this, the type `SYSTEM_VALUE` would just be `string`. 

With the above, you essentially have the same solution as Example 2. But how do we get an object from this? My favorite way to create an object from an array is `Array.reduce()`.

```tsx
const SYSTEM_VALUES_LIST = [
    "value1",
    "value2"
] as const;

const SYSTEM_VALUES = SYSTEM_VALUES_LIST.reduce(
    (acc, systemValue) => ({
        ...acc,
        [systemValue]: systemValue,
    }),
    {}
);
```

But the type of `SYSTEM_VALUES` is now just `{}`, essentially a non-null object. TypeScript is not quite clever enough to sort out what we are doing in the reduce statement. So let's help it out a little. And we can make use of some generics as well to make this code more reusable. 

```tsx
const SYSTEM_VALUES_LIST = [
    "value1",
    "value2"
] as const;

type SYSTEM_VALUE = SYSTEM_VALUES_LIST[number]; // equivalent to "value1" | "value2"

type ConstObjectFromUnion<U extends string> = {
    [Property in U]: Property
};

type SYSTEM_VALUES_TYPE = ConstObjectFromUnion<SYSTEM_VALUE>;

const SYSTEM_VALUES: SYSTEM_VALUES_TYPE = SYSTEM_VALUES_LIST.reduce(
    (acc, systemValue) => ({
        ...acc,
        [systemValue]: systemValue
    }),
    {} as SYSTEM_VALUES_TYPE
);
```

And that is it! May future generations of developers curse you if you add this to your codebase. But to me, this is the briefest way of specifying the list of values, even if the type manipulation is a little excessive. 

## Fibonacci in TypeScript

### Intro

After having seen so many clever manipulations like SQL queries in TypeScript, I wanted to try my hand at a simple test to see if I could get into this kind of thing. 

My first naive attempts at it looked like this. Basically just trying to write the recursive definition into a type. 

```ts
type Fib<N extends number> = N === 0 ? 1 : N === 1 ? 1 : Fib<N-1> + Fib<N-2>;
```

Obviously TypeScript then spat in my face and called me an idiot for trying to do mathematical comparisons inside a type. Trying to fix the errors, I got this.

```ts
type Fib<N extends number> = N extends 0 ? 1 : N extends 1 ? 1 : Fib<N-1> + Fib<N-2>;
```

In TypeScript types, a number extending a constant number literal is the same as an equality check. But this doesnt fix the back half of the equation, arguably the most important part. 

I got stuck at this point for a long time, trying to think of how you could do math inside the type system. I tried Googling a little, but I did not want to give away the answer to myself too early. 

### FizzBuzz

Then I stumbled across a post on Reddit describing how they did FizzBuzz in the type system[^4]. This gave me some idea of how to do mathemetical operations. It did not give away everything, and I still had to figure out quite a bit on my own. 

Lets look at the tools implemented in this example:

```ts
type MyRange<N extends number, Acc extends number[] = []> = Acc["length"] extends N ? Acc : MyRange<N, [...Acc, Acc["length"]]>;
type MyLessThan<A extends number, B extends number, R extends number[] = MyRange<B>> = A extends R[number] ? true : false;
type MyAdd<A extends number, B extends number> = Extract<[...MyRange<A>, ...MyRange<B>]["length"], number>;
```

These are a beast to look at. But let's break it down. `MyRange` is essentially a recursive generic type that constructs an ReadOnlyArray type with elements `[0, ..., N]`. This is the core of all of these math operations, because by extracting the length of that Array, you get a number! By manipulating the length of that Array, you can do math!

Next is `MyLessThan`. Now that we understand `MyRange`, this one is almost trivial. We can check to see if `A` is less than `B` by constructing a `MyRange` type `R` of length `B` and checking if `A` is a valid index of type `R`. Say `A` is 3, and `B` is 5, then we check whether 3 is a valid index of `[0, 1, 2, 3, 4]`. Of course it is!

Lastly, `MyAdd`. The inside part is actually pretty clear. `[...MyRange<A>, ...MyRange<B>]["length"]`. So we just construct a ReadOnlyArray[^5] comprised of the ReadOnlyArray's `MyRange<A>` and `MyRange<B>`. The outer part, `Extract<Result, number>` is actually a bit of black magic to me. I truly do not understand why the inner part is not sufficient, but I can promise you this is necessary for the rest to work. 

All of those utility types were provided by the FizzBuzz example. Everything from here on is my own. 

For Fibonacci, I need to do subtraction, not addition or checking divisibility (as in FizzBuzz). But what is subtraction? Mathematically speaking, we can create a recursive definition of subtraction based on addition. The difference between two numbers `A` and `B` is essentially the number of 1's you need to add to `B` to make it equal to `A`! 

```js
function difference(a, b, n = 0) {
    if (a === b) {
        return n;
    }
    
    return difference(a, b+1, n+1);
}
```

And we can turn that function into a type!

```ts
type MyDifference<A extends number, B extends number, Acc extends number = 0> = A extends B ? Acc : MyDifference<A, MyAdd<B, 1>, MyAdd<Acc, 1>>;
```

Obviously, that function only works (at all) if `A>B` is true. Let's modify the function, then our type.

```js
function difference(a, b, n = 0) {
    if (a === b) {
        return n;
    }
    
    if (a < b) {
        return difference(b, a, n);
    }
    
    return difference(a, b+1, n+1);
}
```

And now apply that same logic to our type.

```ts
type MyDifference<A extends number, B extends number, Acc extends number = 0> = A extends B ? Acc : MyLessThan<B, A> extends true ? MyDifference<A, MyAdd<B, 1>, MyAdd<Acc, 1>> : MyDifference<B, A>;
```

Now that we can do differences (technically not subtraction because we can only tell the magnitude difference between 2 numbers. I leave it as an exercise to the reader to make this thing spit out negative numbers), we can write our Fibonacci type. 

```ts
type Fib<N extends number> = MyLessThan<N, 0> extends true ? 0 : N extends 0 ? 1 : N extends 1 ? 1 : MyAdd<Fib<MyDifference<N, 1>>, Fib<MyDifference<N, 2>>>;
```

We at least have a little type safety in that this type is 0 for all `N<0`. Using the TS playground, I found the maximum N for which the TS system would not give up was 16. 

Wanna try it yourself? Try out my [TS Playground example here](https://www.typescriptlang.org/play?#code/FAFwngDgpgBAsmASgQwHYHMoB4ByMoAeIUqAJgM4yoCuAtgEZQBOANDAIIDGn+RJFVOoyYBtALowAvDHEA+KR24iARABsS6EAAtlEwsTKU8AfkU8AXPCRpMuNiIB0TrpzYuV6jNt1jZAblBIWAQAGShycgAVLTQsdl4DARoGZjYAIQT+SmThNkRMw0EU0QlpBBQMbDTZeWl4-SyYRBEc5glTECZqWEsAM2RVcigA8GgrdlJSOIKkoVSYDIbC1qZamABRIiZkThAsRydym2x2WTYnByPKrGqxDw1vMTYV-0CxhAARAEte3uYSTgnGbZOasBbAoq5MwQlYKAAMa3qfEKGVMLhgllC4SiMVQNzc8iWAk63Rgpk+Pz+TABJzYCAmUzSbAAjGdxpM4twWTUMVZvr9-qhAfiOK83rAAGJfei4GGgtZYiLRWI4NgIiEk2CmOG8vBEyg60zM3UQ41G3n0jlSmUUgXUoXYVUwVls61YW1Uml2GAAJhqYtGsEi4RAcGQBAUbuZADZ-EA). 

## Conclusion
Wow. This has been a whirlwind of a post. I hope you enjoyed this journey as much as I did. I don't know how much I will continue to delve into this world of type gymnastics, but I will leave you some resources here if you want to do so:

- [An introduction to programming in the TypeScript type system](https://www.zhenghao.io/posts/type-programming)
- [Implementing Arithmetic in the TypeScript type system](https://itnext.io/implementing-arithmetic-within-typescripts-type-system-a1ef140a6f6f)
- [Insane TypeScript gymnastics projects](https://www.learningtypescript.com/articles/extreme-explorations-of-typescripts-type-system)

[^1]: https://github.com/codemix/ts-sql
[^2]: https://github.com/ronami/HypeScript
[^3]: This is all because the TypeScript system is Turing-complete. So technically you could do implement any programming concept in the type system. 
[^4]: https://www.reddit.com/r/typescript/comments/x3hq1y/typelevel_fizzbuzz/
[^5]: Really a tuple, but essentially the same.
