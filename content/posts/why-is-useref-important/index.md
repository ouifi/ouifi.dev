---
title: "Why is `useRef` Important?"
date: "2022-11-15"
publishDate: "2022-11-15"
Toc: true
description: "In this article we explore why React's `useRef` is so important by looking at alternative attempts to achieve the same effect."
tags: 
  - react
  - tech
  - typescript
keywords:
  - hooks
  - react
---

# Why is `useRef` important?

## Intro

Because we need stable references to some data or DOM object. Because React is inherently a system of data trickling down a tree, where we must treat objects and data as immutable. `useRef` is an escape hatch into the world of mutability. 

This article will not go into depth about how the `useRef` hook actually works. Frankly I do not understand nor feel the need to attempt to. If you do, seek elsewhere. Plenty of articles exist to fill this need, or you can try your hand at the React source code.[^1] 

Instead we will study how might one have tried to solve the same problems as `useRef` if it did not exist.

As always, the examples are [available on my GitHub page.](https://github.com/ouifi/why-is-useref-important-examples)

## Correct `ref` usage

`useRef` is important because each copy of the component gets its own ref, the same one every time.

Just for reference, let's take a look at how refs should be properly used.

{{< code language="typescript" title="src/components/CorrectRef.tsx" id="1" isCollapsed="false" >}}

import { useRef } from "react";

const colors = [
    "red", 
    "blue",
    "green",
    "yellow",
    "pink",
]

export const CorrectRefExample = () => {
    const buttonRef = useRef<HTMLButtonElement>(null);
    return <button ref={buttonRef} style={{
        padding: 20,
        fontSize: "2rem"
    }}
    onClick={
        () => {
            buttonRef.current?.setAttribute("class", colors[Math.round(Math.random() * 4)]);
        }
    }>
        Click me!
    </button>
}

{{< /code >}}

We also need to add the following to our `index.css` file so that the classnames apply a styling to the component. We will just leave this in here for all the examples. 

{{< code language="css" title="src/index.css" id="2" isCollapsed="false" >}}

.red {
  background-color: red;
}

.blue {
  background-color: blue;
}

.yellow {
  background-color: yellow;
}

.pink {
  background-color: pink;
}

.green {
  background-color: green;
}

{{< /code >}}

Inserting it into our `App.tsx`:

{{< code language="typescript" title="src/App.tsx" id="3" isCollapsed="false" >}}

import { FileRefExample } from './components/FileRef';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <CorrectRefExample/>
      </header>
    </div>
  );
}

export default App;

{{< /code >}}

And now it works!

{{< video "images/correctref.webm" >}}

## Why not use a file-scope const?

A thought occurred to me a while ago, why does one need `useRef` when a file-scope const object with a `current` field would provide the same level of mutability?

Let's try it out. 

{{< code language="typescript" title="src/components/FileRef.tsx" id="4" isCollapsed="false" >}}

type ButtonRef = {
    current: HTMLButtonElement | null
}

const ref: ButtonRef = {
    current: null
};

const colors = [
    "red", 
    "blue",
    "green",
    "yellow",
    "pink",
]

export const FileRefExample = () => {
    return <button 
        ref={ref} 
        style={{padding: 20, fontSize: "2rem" }}
        onClick={
            () => {
                ref.current?.setAttribute("class", colors[Math.round(Math.random() * 4)]);
            }
        }
    >
        Click me!
    </button>
}

{{< /code >}}

Let's add that component to our `App.tsx`.

{{< code language="typescript" title="src/App.tsx" id="5" isCollapsed="false" >}}

import { FileRefExample } from './components/FileRef';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <FileRefExample/>
      </header>
    </div>
  );
}

export default App;

{{< /code >}}

Now let's see how this works!

{{< video "images/fileref.webm" >}}

Looks good to me! Man, I love this component so much, I am going to add a second one. 

{{< code language="typescript" title="src/App.tsx" id="6" isCollapsed="false" >}}

import { FileRefExample } from './components/FileRef';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <FileRefExample/>
        <FileRefExample/>
      </header>
    </div>
  );
}

export default App;

{{< /code >}}

{{< video "images/filerefbad.webm" >}}

**Oh No**

Our code has done exactly as we instructed. We created a single ref, which was passed to the both components. The one that mounts last[^2] gets control, and clicking on either button acts on the ref to manipulate the class of the button. 

This code actually works just fine as long as your component only gets used once in the DOM. This might be the case for a high level component like `App` or a `NavBar`. But there is not really any good reason to do this when `useRef` would accomplish the same thing in the single-use or multi-use examples. 

This is why `useRef` is so important. Because `useRef` works fine! For example:

{{< code language="typescript" title="src/App.tsx" id="7" isCollapsed="false" >}}

import { FileRefExample } from './components/FileRef';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <CorrectRefExample/>
        <CorrectRefExample/>
      </header>
    </div>
  );
}

export default App;

{{< /code >}}

Produces:

{{< video "images/correctref2.webm" >}}

Each button gets its own independent `ref` that is not intertwined with any other component. 

## Conclusion

If there is one thing to take away from this article, it's that the React developers were pretty clever. The flawed way we implemented refs is *kind of* like the original implmementation, way back before React 16.4[^3] Back then, your ref was identified by a magic string, attached to the class object, instead of the file scope. This was eventually superseded by callback refs, then `React.createRef`, and now in the modern era, `useRef`. 

The other lesson here is to read the documentation. I can easily imagine a junior developer attempting this kind of naive solution. Thankfully it is a pretty buggy solution, so it shouldn't last long in anyone's repo. 


[^1]: [Good luck with that](https://github.com/facebook/react/blob/8e2bde6f2751aa6335f3cef488c05c3ea08e074a/packages/react/src/ReactHooks.js#L116)
[^2]: This is likely a race condition, but whichever is deeper in the DOM tree will almost certainly end up with control. 
[^3]: https://reactjs.org/docs/refs-and-the-dom.html#legacy-api-string-refs