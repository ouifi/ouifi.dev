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

Using a const enum:

```tsx
export enum SYSTEM_VALUES {
    value1 = "value1",
    value2 = "value2",
    value3 = "value3"
}

type ComponentProps = {
    systemValue: SYSTEM_VALUES
}

export const Component = ({ systemValue }: ComponentProps) => {
    // use the systemValue prop
    return <div>{systemValue}</div>;
};

// Then when you use the component...
export const Example1 = () => {
    return <Component systemValue={SYSTEM_VALUES.value1}/>
}
```

Using a union of strings

```tsx
export type SYSTEM_VALUES = 
    | "value1"
    | "value2"
    | "value3"

type ComponentProps = {
    systemValue: SYSTEM_VALUES
}

export const Component = ({ systemValue }) => {
    // use the systemValue prop
    return <div>{systemValue}</div>;
}

// use the component
export const Example2 = () => {
    return <Component systemValue="value1"/>
}
```

I wont go into the virtues of either of these approaches, and the real example was obviously more complex. But this did make me think: "Surely there is a way to have a strongly typed way of referencing a list of strings by value or by the variable name". 

### The Solution

[^1]: https://github.com/codemix/ts-sql
[^2]: https://github.com/ronami/HypeScript
[^3]: This is all because the TypeScript system is Turing-complete. So technically you could do implement any programming in TypeScript. 