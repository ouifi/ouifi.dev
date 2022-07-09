---
title: "React API Hooks (Part 2)"
date: "2022-07-15"
publishDate: "2022-07-15"
Toc: true
tags:
  - react
  - tech
keywords: 
  - development
  - react
  - hooks
  - api
description: "More advanced usage of React Hooks to drive your API."
---

# Intro

In the last article, I described how to get a good bang for your buck out of React hooks to fetch API data. This also included tying in the authentication state with automatic sign-out features. 

In this article, I want to address one of the exercises I left for the reader. In short, deduplicate the API calls which would be made in the case of many copies of our hooks being rendered all at once. 

Just as before, the complete examples are [available on my GitHub page](https://github.com/ouifi/react-api-hooks-examples/tree/v2). This is just on a `v2` branch of the same solutions repo from the frst article.

# Recap

Let's review where we left off. 

{{< code language="typescript" title="src/hooks/useApi.ts" id="1" isCollapsed="false" >}}
import { useCallback } from 'react';
import API from 'services/api';

export const useApi = () => {
  const get = useCallback(
    async (url: string | Request, opts?: Record<string, unknown>) => {
      return API.get(url, opts);
    },
    []
  );

  const post = useCallback(
    async (url: string | Request, body: Record<string, unknown>, opts?: Record<string, unknown>) => {
      return API.post(url, body, opts);
    },
    []
  );

  return { get, post }
}
{{< /code >}}

And 

{{< code language="typescript" title="src/hooks/useChuckNorrisApi.ts" id="2" isCollapsed="false" >}}
import { useState, useEffect } from 'react';
import { useApi } from 'hooks/useApi';
import API from 'services/api';

export type ChuckNorrisJoke = {
  id: number;
  joke: string
}

export const useChuckNorrisApi = () => {
  const { get } = useApi();

  const [data, setData] = useState<ChuckNorrisJoke>()

  useEffect(
    () => {
      get("http://api.icndb.com/jokes/random")
        .then(API.isOk)
        .then(API.asJSON)
        .then(
          (d) => {
            setData(d.value);
          }
        );
    },
    [get]
  );

  return { joke: data };
}
{{< /code >}}

This solution works great for the simple use cases! If you are just rendering the odd one-off Chuck Norris joke, this works perfectly. 

# The Problem

The problem begins when you need to render this data to multiple places. For example, I have written an API hook in another project that calls for a list of fixed data to be rendered as options in a listbox. But there are multiple sub-forms on the same page, and so this listbox gets rendered multiple times. For each render, the hook is invoked and a network call is made. 

To see this in our Chuck Norris example, I duplicated the `Example` component in the `App`. 

{{< code language="typescript" title="src/App.tsx" id="3" isCollapsed="false" >}}
import React, { useContext } from 'react';
import logo from './logo.svg';
import './App.css';
import { QueryClient, QueryClientProvider } from 'react-query';
import { Example1 } from 'components/Example1';
import { AuthContext, AuthProvider } from 'context/authContext';

const Protector = ({ children }: { children: React.ReactNode }) => {
  const { isAuthenticated, attemptSignIn, triggerSignOut } = useContext(AuthContext);

  return <>
    {
      isAuthenticated
        ? <>
          {children}
          <button style={{ backgroundColor: "yellow", color: "#61dafb", padding: 10, fontSize: "larger", borderRadius: "5px", border: "none" }} onClick={() => triggerSignOut()}>
            Log Out
          </button>
        </>
        : <>
          <span>Click the button to see cool Chuck Norris facts!</span>
          <button style={{ backgroundColor: "#61dafb", color: "yellow", padding: 10, fontSize: "larger", borderRadius: "5px", border: "none" }} onClick={() => attemptSignIn()}>
            Log In
          </button>
        </>
    }
  </>;
}

const queryClient = new QueryClient();

function App() {
  return (
    <AuthProvider>
      <div className="App">
        <header className="App-header">
          <Protector>
            <img src={logo} className="App-logo" alt="logo" />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
            <Example1 />
          </Protector>
        </header>
      </div>
    </AuthProvider>
  );
}

export default App;

{{< /code >}}

And this is the result:

{{< figure src="images/pyramid_of_death.png" title="No!" >}}

As you can see, it does render a ton of Chuck Norris jokes. Each one is different. This is qualitative proof that each hook rendered produces an independent network request (Joke). The objective proof is in the graph on the right. Each network request kicks off at the same time, when the page first loads. Then they compete to get completed, creating a pyramid cascade of doom. 

At the end of the previous article, I clued in the reader that the solution may already exist in the `React-Query` library. I had heard of this library and read the docs. It was impressive all the problems it claimed to solve. I was fairly confident this problem would be fairly trivial using `React-Query`. Turns out, I was right.



