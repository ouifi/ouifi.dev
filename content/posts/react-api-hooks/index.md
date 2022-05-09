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

src/api/data.js
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

Writing the boilerplate for API calls and user app authentication all in one carries huge benefits. By tying them all together and layering them in a smart way, you can write clean, efficient frontend boilerplate. 

First, we will write a simple abstraction


[^1]: https://tsh.io/state-of-frontend/#over-the-past-year-which-of-the-following-libraries-have-you-used-and-liked
[^2]: This is probably also in large part due to `axios` being cross-platform, so developers could use it in their frontend clients, standalone JS scripts which pull data from the web, or in backend servers when integrating with a third party API. Fetch only got native support in NodeJS core in v18, released spring 2022. 