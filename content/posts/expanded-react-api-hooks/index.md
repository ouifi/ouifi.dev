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

# React API Hooks, Part 2

## Intro

In [the last article](/posts/react-api-hooks), I described how to get a good bang for your buck out of React hooks to fetch API data. This also included tying in the authentication state with automatic sign-out features. 

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

## The Problem

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

As you can see, it does render a ton of Chuck Norris jokes. Each one is different, and while the page is loading, the content loads in a stuttering manner. This is qualitative proof that each hook rendered produces an independent network request (Joke). The objective proof is in the graph on the right. Each network request kicks off at the same time, when the page first loads. Then they compete to get completed, creating a pyramid cascade of doom. 

At the end of the previous article, I clued in the reader that the solution may already exist in the `React-Query` library. I had heard of this library and read the docs. It was impressive all the problems it claimed to solve. I was fairly confident this problem would be fairly trivial using `React-Query`. Turns out, I was right.

## The Solution

`React-Query` is currently in flux, as it is about to become TanStack Query when v4 releases. This solution utilizes v3, as it is the latest production-ready version at time of writing. I will update this article when TanStack Query officially releases. 

What will `React-Query` do for us? The main draw is that it is an opinionated data-management library which brings a lot of the good parts of `Apollo/GQL` back to the REST world. The main feature we will rely on for this article is the Caching. For every API request made using `React-Query`, the library will check its cache to see if the right data is already stored on the client! It deduplicates these API requests. It has tons more features, but this simple one will massively improve the performance of data-fetching heavy applications. 

Starting with the repo as it ended in the last article, we add `React-Query` as a dependency.

```bash
npm i react-query
```

Following the simplest [example in the docs](https://react-query-v3.tanstack.com/guides/queries), we end up with this boilerplate:

{{< code language="typescript" title="src/App.tsx" id="4" isCollapsed="false" >}}
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
    <QueryClientProvider client={queryClient}>
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
            </Protector>
          </header>
        </div>
      </AuthProvider>
    </QueryClientProvider>
  );
}

export default App;

{{< /code >}}

Super simple! We wrap our top level code with the `QueryClientProvider` and now we can get off to the races. Let's look next at how we need to modify our API hook code we wrote in the last article. In order to make our API hooks more composable, I am not going to add `React-Query` to our `useApi.ts` or `useProtectedApi.ts` code, as it is essentially a wrapper around `fetch` with our Auth boilerplate. Since `React-Query` is more about how we access data, it makes more sense to add it to each API hook. Then each API hook can provide its own configuration to `React-Query` as needed. We will pass our own fetching function to `useQuery`, and observe the results. 

{{< code language="typescript" title="src/services/ChuckNorris.ts" id="5" isCollapsed="false" >}}
import { useCallback } from 'react';
import { useQuery } from 'react-query';
import { useApi } from 'hooks/useApi';
import API from 'services/api';

export type ChuckNorrisJoke = {
  id: number;
  joke: string
};

export const useChuckNorrisApi = () => {
  const { get } = useApi();

  const fetchChuckNorrisJoke = useCallback(
    () => {
      return get('http://api.icndb.com/jokes/random')
        .then(API.isOk)
        .then(API.asJSON)
        .then(
          (d) => d.value
        );
    },
    [get]
  );

  return useQuery<unknown, unknown, ChuckNorrisJoke>("chucknorris", fetchChuckNorrisJoke);
};

{{< /code >}}

Of note: 1) We pass the type of the result data as the 3rd generic parameter of `useQuery`. 2) I have wrapped the `fetch` call with `useCallback`. This is probably not strictly necessary but it helps to stabilize the reference to this arrow function. 3) We are doing all of the error handling for this API call in the callback, as [recommended in the docs](https://react-query-v3.tanstack.com/guides/query-functions#usage-with-fetch-and-other-clients-that-do-not-throw-by-default).

Because we are passing back the `useQuery` result directly, we need to adjust how our components consume this hook by destructuring the `data` field. 

{{< code language="typescript" title="src/components/Example1.ts" id="5" isCollapsed="false" >}}
import React from 'react';
import { useChuckNorrisApi } from 'services/ChuckNorris';

export const Example1 = () => {
  const { data: joke } = useChuckNorrisApi();

  return <p>
    {
      joke && `Joke #${joke.id}: ${joke.joke}`
    }
  </p>
};
{{< /code >}}

Lets look at the result!

{{< figure src="images/dot_of_joy.png" title="Yes!" >}}

Yay! Our `App` ends up only making a single API call, and each rendered `Example` component gets the same copy of the data. 

The only other adjustment to make is to suppress one of the (generally) nice features of `React-Query`. The natural behavior of the library is to refresh data whenever the user leaves and returns to the application, such as by clicking on another tab or window, then back. 

{{< figure src="images/cache_refresh.gif" title="Yes!" >}}

We can suppress this by playing with the `staleTime` parameter of our `useQuery` call. Setting it to `Infinity` should prevent this refresh from happening by declaring that this data will never become stale to the user.[^1]

The addition goes here:

```typescript
return useQuery<unknown, unknown, ChuckNorrisJoke>("chucknorris", fetchChuckNorrisJoke, { staleTime: Infinity });
```

## Conclusion

By utilizing the premier feature of `React-Query`, we can drastically improve the process of our application loading data. This use case demonstrates the utility of `React-Query` on loading highly duplicated or repeated data. Obviously this will not be a silver bullet for improving performance. We could have written a caching utility which would probably work for the same use case. But `React-Query` is a fantastic open source project which is probably better than anything I could come up with. Plus, it has so much more to offer in the way of data communication that I have not even touched on in this brief article. 

[^1]: Chuck Norris jokes are indeed timeless, much like Chuck Norris himself. 