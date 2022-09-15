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

[^1]: https://github.com/codemix/ts-sql
[^2]: https://github.com/ronami/HypeScript
[^3]: This is all because the TypeScript system is Turing-complete. So technically you could do implement any programming concept in the type system. 
