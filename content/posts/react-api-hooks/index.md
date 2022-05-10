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

When I worked in my first role as a frontend developer, we basically had the worst possible way of managing API calls. I came into the project when it was about 1/3 done. A lot of the boilerplate code had been written, and since it was my first frontend job, I did not know any better to improve it. So I respected the solo senior developer's choices and moved on. This is basically how the code base was structured:

## The Problem

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

```js
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
```

src/pages/savedItems.js
```js
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
```

Yep. Class-based components and `axios`. To be fair, I think the Hooks API had only been released a few months before I joined the team. `Fetch` had been out and supported in Chrome since 2015 (Chrome v42), but was not commonly used. Axios definitely had (and continues to have [^1]) a large market share of client-server communications in JS[^2]. 

The senior developer also had not included much (if any) boilerplate code around user authentication. It was basically not included in the app at all. When I started working on it as we neared our deadline, I wanted to follow best practice. But all of the guides I consulted used React functional components, Hooks, and Context to implement auth. So I started writing my own boilerplate that would completely revamp how the app would run at its core. 

## The Solution

Writing the boilerplate for API calls and user app authentication all in one carries huge benefits. By tying them all together and layering them in a smart way, you can write clean, efficient frontend boilerplate. Our starting point will be a freshly initialized repo using `create-react-app`. 

### Abstract Fetch

First, we will write a simple abstraction over the native Fetch Web API. At minimum, lets implement GET, POST, and a couple of helper methods. 

src/services/api.ts
```js
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
}

export default API;
```

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
  )
```

\* Chef's Kiss \*

Now we have a terse but effective way to write API requests in our application. Even if we stopped here, we would have a DRY-er way of using `fetch`. 

### Hook-ify our API Requests

This boilerplate code does not yet serve a purpose in and of itself, but it will become clear later why we needed it. 

src/hooks/useApi.ts
```js
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
```

Now we can access these API call methods from within React hooks like this:

src/hooks/useExample1.ts
```js
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
```

Now, let's add some practical stuff and use a concrete example

src/hooks/useChuckNorris.ts
```js
import { useState, useEffect } from 'react';
import { useApi } from 'hooks/useApi';
import API from 'services/api';

export type ChuckNorrisJoke = {
  id: number;
  joke: string
}

export const useChuckNorris = () => {
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
```

Then we consume this hook in a component

src/components/Example1.tsx
```js
import React from 'react';
import { useChuckNorris } from 'hooks/useChuckNorris';

export const Example1 = () => {
  const { joke } = useChuckNorris();

  return <p>
    {
      joke && `Joke #${joke.id}: ${joke.joke}`
    }
  </p>
}
```

Adding this component to `src/App.tsx`, we get this!

{{< figure src="images/chucknorris.png" title="Free Chuck Norris Jokes!" >}}

I quite like this pattern for several reasons. 1) It is very little code, so it is not a large burden to write a hook for each model in your API. 2) The type of the API data is highly local to its retrieval. 3) Because they are hooks, they are composable! You could write a hook C, which is a composite hook of hooks/models A and B. But we can do even better.
object
### Auth Boilerplate

I will not go into too much detail on the implementation of the authentication code. How you implement the specifics of authenticating to your API is completely up to you. Fill in the gaps where appropriate. This example is NOT production-ready, but its a skeleton of a good auth management component. 


[^1]: https://tsh.io/state-of-frontend/#over-the-past-year-which-of-the-following-libraries-have-you-used-and-liked
[^2]: This is probably also in large part due to `axios` being cross-platform, so developers could use it in their frontend clients, standalone JS scripts which pull data from the web, or in backend servers when integrating with a third party API. Fetch only got native support in NodeJS core in v18, released spring 2022. 