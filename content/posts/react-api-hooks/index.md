---
title: "React API Hooks"
date: "2022-05-15"
publishDate: "2022-05-15"
Toc: true
tags:
  - react
  - tech
keywords: 
  - development
  - react
  - hooks
  - api
description: "Use React Hooks to drive how your frontend consumes an API."
---

# React API Hooks

## Intro

Hooks are all the rage now in React applications. Everyone knows how to build them and use them to encapsulate common logic and reuse it. But can this pattern be applied to our API calls? In addition, how can we ensure that a user whose authentication has expired (either their token is no longer valid or their session has ended) is properly shown the door? (So to speak.) 

The finished examples developed in this article are [available on my GitHub page](https://github.com/ouifi/react-api-hooks-examples).

## Background and Problem Statement

When I worked in my first role as a frontend developer, we basically had the worst possible way of managing API calls. I came into the project when it was about 1/3 done. A lot of the boilerplate code had been written, and since it was my first frontend job, I did not know any better to improve it. So I respected the solo senior developer's choices and moved on. This is basically how the code base was structured:

{{< code language="javascript" title="src/api/data.js" id="1" isCollapsed="false" >}}
import axios from 'axios';

async function getSavedItems(user_id) {
  return axios.get("/savedItems?user_id=" + user_id)
    .then(
      (res) => {
        return res.data;
      }
    )
}

// ...

async function getListItems() {
  return axios.get("/listItems")
    .then(
      (res) => {
        return res.data;
      }
    )
}

// ...

export default {
  getSavedItems,
  //...
  getListItems,
}
{{< /code >}}


{{< code language="javascript" title="src/pages/savedItems.js" id="2" isCollapsed="false" >}}
import React from 'react';
import { getSavedItems } from '@api/data';

class SavedItems extends React.PureComponent {
  constructor() {
    this.state = {
      savedItems: null
    };

    this.super();
  }

  componentDidMount() {
    getSavedItems(1)
      .then(
        (data) => {
          this.setState({
            savedItems: data
          })
        }
      )
  }

  render() {

    if (!this.state.savedItems) {
      return "Loading...";
    }

    if (this.state.savedItems.length === 0) {
      return "No data to show";
    }

    return <>
      {/*
        Render a table with all the saved items
      */}
    </>
  }
}
{{< /code >}}

Yep. Class-based components and `axios`. To be fair, I think the Hooks API had only been released a few months before I joined the team. `Fetch` had been out and supported in Chrome since 2015 (Chrome v42), but was not commonly used. Axios definitely had (and continues to have [^1]) a large market share of client-server communications in JS[^2]. 

The senior developer also had not included much (if any) boilerplate code around user authentication. It was basically not included in the app at all. When I started working on it as we neared our deadline, I wanted to follow best practice. But all of the guides I consulted used React functional components, Hooks, and Context to implement auth. So I started writing my own boilerplate that would completely revamp how the app would run at its core. 

## The Solution

Writing the boilerplate for API calls and user app authentication all in one carries huge benefits. By tying them all together and layering them in a smart way, you can write clean, efficient frontend boilerplate. Our starting point will be a freshly initialized repo using `create-react-app`. 

### Abstract Fetch

First, we will write a simple abstraction over the native Fetch Web API. At minimum, lets implement GET, POST, and a couple of helper methods. 

{{< code language="typescript" title="src/services/api.ts" id="3" isCollapsed="false" >}}
async function asJSON(res: Response) {
  return res.json();
}

async function isOk(res: Response) {
  if (res.ok) {
    return res;
  } else {
    throw new Error(res.statusText);
  }
}

async function get(url: string | Request, opts?: Record<string, any>) {
  const trueOptions = {
    method: "GET",
    ...opts
  };

  return fetch(url, trueOptions);
}

async function post(url: string | Request, body: Record<string, any>, opts?: Record<string, any>) {
  const trueOptions = {
    headers: {
      "Content-Type": "application/json",
    },
    method: "POST",
    body: JSON.stringify(body),
    ...opts
  };

  return fetch(url, trueOptions);
}

const API = {
    get, post, isOk, asJSON
};

export default API;
{{< /code >}}

Let's look at two equivalent examples of using `fetch`. One with this abstraction, and one without.

Without:
```js
fetch("/api/data")
  .then(
    (res) => {
      if (res.ok) {
        return res;
      } else {
        throw new Error(res.statusText)
      }
    }
  )
  .then(
    (res) => {
      return res.json();
    }
  )
  .then(
    (data) => {
      console.log(data);
    }
  );
```

With:
```js
import API from 'services/api'
API.get("/api/data")
  .then(API.isOk)
  .then(API.asJSON)
  .then(
    (data) => {
      console.log(data);
    }
  );
```

\* Chef's Kiss \*

Now we have a terse but effective way to write API requests in our application. Even if we stopped here, we would have a DRY-er way of using `fetch`. 

### Hook-ify our API Requests

This boilerplate code does not yet serve a purpose in and of itself, but it will become clear later why we needed it. 

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

Now we can access these API call methods from within React hooks like this:

src/hooks/useExample1.ts
{{< code language="typescript" title="src/hooks/useExample1.ts" id="4" isCollapsed="false" >}}
import { useState, useEffect } from 'react';
import { useApi } from 'hooks/useApi';
import API from 'services/api';

export const useExample = () => {
  const { get } = useApi();

  const [data, setData] = useState()

  useEffect(
    () => {
      get("/api/example")
        .then(API.isOk)
        .then(API.asJSON)
        .then(
          (d) => {
            setData(d);
          }
        );
    },
    [get]
  );

  return data;
}
{{< /code >}}

Now, let's add some practical stuff and end up with a concrete example

{{< code language="typescript" title="src/hooks/useChuckNorrisApi.js" id="5" isCollapsed="false" >}}
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

This hook is not fetching from a protected API. The Internet Chuck Norris Database is an open, free API for anyone to use. 

We can consume this hook in a component

{{< code language="typescript" title="src/components/Example1.tsx" id="6" isCollapsed="false" >}}
import React from 'react';
import { useChuckNorrisApi } from 'hooks/useChuckNorrisApi';

export const Example1 = () => {
  const { joke } = useChuckNorrisApi();

  return <p>
    {
      joke && `Joke #${joke.id}: ${joke.joke}`
    }
  </p>
}
{{< /code >}}

Adding this component to `src/App.tsx`, we get this!

{{< figure src="images/chucknorris.png" title="Free Chuck Norris Jokes!" >}}

I quite like this pattern for several reasons. 1) It is very little code, so it is not a large burden to write a hook for each model in your API. 2) The type of the API data is highly local to its retrieval. 3) Because they are hooks, they are composable! You could write a hook C, which is a composite hook of hooks/models A and B. But we can do even better.

### Auth Boilerplate

I will not go into too much detail on the implementation of the authentication code. How you implement the specifics of authenticating to your API is completely up to you. Fill in the gaps where appropriate. This example is NOT production-ready, but its a skeleton of a good auth management component. 

{{< code language="typescript" title="src/context/AuthContext.tsx" id="7" isCollapsed="false" >}}
import React, { createContext, useState } from "react";

const AuthContext = createContext({
  attemptSignIn: () => { },
  triggerSignOut: () => { },
  isAuthenticated: false
});

const AuthProvider = ({ children }: { children: React.ReactNode }) => {

  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const attemptSignIn = () => {
    setIsAuthenticated(true);
  }

  const triggerSignOut = () => {
    setIsAuthenticated(false);
  }

  return <AuthContext.Provider value={{ attemptSignIn, triggerSignOut, isAuthenticated }}>
    {children}
  </AuthContext.Provider>;
};

export {
  AuthContext,
  AuthProvider
}
{{< /code >}}

In a more robust implementation, the `attemptSignIn` and `triggerSignOut` methods would include some kind of API call to the authentication endpoint to sign in and sign out. There would also be a little more complex status *about* the signed in user, like their name or email address.[^3]

With this in place, we can apply some slight changes to our `App.tsx` file and get a page that starts to feel like something we could see in a real web app. 

{{< code language="typescript" title="src/App.tsx" id="8" isCollapsed="false" >}}
import React, { useContext } from 'react';
import logo from './logo.svg';
import './App.css';
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

function App() {
  return (
    <AuthProvider>
      <div className="App">
        <header className="App-header">
          <Protector>
            <img src={logo} className="App-logo" alt="logo" />
            <Example1 />
          </Protector>
        </header>
      </div>
    </AuthProvider>
  );
}

export default App;
{{< /code >}}

Running this code, we get:

{{< figure src="images/loggedout.png" title="Free Chuck Norris Jokes!" >}}

And after pressing the button:

{{< figure src="images/loggedin.png" title="Free Chuck Norris Jokes!" >}}

### Hooking it all together

There is one more thing we can do to really tie this all up. Consider what we have. We have a hook which provides us with API call primitives (`get` and `post`). We have a context/hook system which lets us manipulate the logged in/logged out status of our app. And we have an API hook which provides us with data. Let's revisit our `useApi` hook. Why did we make this a hook? It doesnt add anything really on top of the `services/api.ts` primitives. 

Let's change that. Because we wrote this in a hook, we can consume other hooks in it. Let's build a new hook called `useProtectedApi`

{{< code language="typescript" title="src/hooks/useProtectedApi.ts" id="9" isCollapsed="false" >}}
import { useCallback, useContext } from 'react';
import { AuthContext } from 'context/authContext';
import API from 'services/api';

export const useProtectedApi = () => {
  const { triggerSignOut } = useContext(AuthContext);

  const protectedCall = useCallback(
    async (p: Promise<Response>) => {
      const response = await p;
      if (response.status === 401 && response.statusText === "Unauthorized") {
        triggerSignOut();
      }
      return response;
    },
    [triggerSignOut]
  );

  const get = useCallback(
    async (url: string | Request, opts?: Record<string, unknown>) => {
      return protectedCall(API.get(url, opts));
    },
    [protectedCall]
  );

  const post = useCallback(
    async (url: string | Request, body: Record<string, unknown>, opts?: Record<string, unknown>) => {
      return protectedCall(API.post(url, body, opts));
    },
    [protectedCall]
  );

  return { get, post }
}
{{< /code >}}

And boom! Now we have, natively built-in to our React application logic, an Auto-Sign Out feature. Whenever the frontend makes an API call to an auth-protected endpoint, this hook will check for a 401 status and completely kick the user out, before they can do anything else. This is perfect for idle users, who have left the screen open. When they return to the app and load any new page that requires a protected API call, they will be shown the door.

## Conclusion and Discussion

I like this pattern because it encapsulates a lot of logic into relatively few lines of code. In addition, there is flexibility. You could write a perfectly good API hook which did not consume the `useProtectedApi` hook. But having this as a sensible starting point for writing API hooks makes development a breeze. 

There are 3 exercises I will leave to the reader, in increasing levels of difficulty.

1) Implement a more complex `useChuckNorrisApi` hook. According to the [documentation,](http://www.icndb.com/api/)if you supply the `firstName` and `lastName` parameters in the query string, you can insert the name of any character you want! Modify the application we built to take in user input and re-fetch the API with those dynamic parameters. How will you pass them to the `useChuckNorrisApi` hook? Consider adding a dependency array to the `useChuckNorrisApi` hook.

2) Deduplicate these API calls. Consider a very long page with multiple address forms being rendered at once, each with a dropdown for the Country. If each of those dropdowns renders a `useCountryApi` hook, then the API end point will be re-fetched one time for each instance of the dropdown. Now you are wasting network resources and potentially slowing down your app waiting for all of these duplicate dropdowns to populate! Can we deduplicate them? Hint: consult the `react-query` package. 

3) Flesh out and implement the `AuthContext`. Consider what logic you would include in the `attemptSignIn` and `triggerSignOut` methods. We initalized the `isAuthenticated` variable to always be `false`. Is this a good assumption? How do you determine if the user is signed in on page load?

Again, the finished examples of everything we worked on here are [available on my GitHub page](https://github.com/ouifi/react-api-hooks-examples). Consider forking from there to work on the problems above!


[^1]: https://tsh.io/state-of-frontend/#over-the-past-year-which-of-the-following-libraries-have-you-used-and-liked
[^2]: This is probably also in large part due to `axios` being cross-platform, so developers could use it in their frontend clients, standalone JS scripts which pull data from the web, or in backend servers when integrating with a third party API. Fetch only got native support in NodeJS core in v18, released spring 2022. 
[^3]: For a robust AuthContext implementation, see Ryan Chenkie's course on building secure React apps