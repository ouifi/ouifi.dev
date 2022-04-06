---
title: "Choosing Good Primary Keys"
date: "2022-04-15"
publishDate: "2022-04-15"
tags:
  - tech
  - sql
  - databases
Toc: true
description: "A guide on choosing good primary keys in your database-driven application."
---

# Choosing Good Primary Keys

## Introduction

Before getting into the meat of the discussion, let us first address why you are here. Not in the cosmic sense, or in the "why did you pick this article from the 14th page of Google search results?" way. You are here because you had the same thoughts I did. Let's zoom in on the first day I sat down to write my first web-application.

> Alright, I have got my Git repos, let me start writing API endpoints. First I will write a `/users/` set of endpoints.Most applications have users. And now to write the `user` table in the database. Ok, easy; let's start with `id`, `name`, `email`. Nice! Making great progress. Let's pick our data types. `name` has gotta be a string, so let's go with `TEXT`.[^1]. `email` should be `TEXT` too, and we will enforce the format in the API; no problem. `id` should probably just be an integer, starting with 1. And make it the primary key too! Then we can reference it in other tables. Hm, before I get too deep, let me just Google this to see what other people think. 

And that is how I found 10 articles with 10 opinions about selecting a primary key.

## What's wrong with integer?

This was my first question. Almost every time I have written database/SQL code in my (short) career, I default to having an integer `id` field in every table. This forces the table to conform to 3rd normal form in most practical cases. Using an integer also means it is easy to tell (in general), the order in which rows were added. The table declaration also looks very clean. Using PostgreSQL:

```
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name TEXT,
  email TEXT
);
```

Nice and simple.[^2] So what is wrong with this? Nothing really. For simple use cases, using an integer `id` is probably fine. But I chose this example for a reason. 

## What is wrong with integer

Let's go back to our example of building a database-backed application, in the REST API. When building out the `/users/` endpoints, consider how we will reference our users. We have to include `id` somewhere in our request in order for the API to know which user we want. For sake of argument, let us say endpoints in this API use the `/users/:id` convention. 

Herein lies the issue. By including an integer `id` field in the URL, we have just opened up an entire avenue for cyber attackers to extract data from our application. By enumerating `id` digits, (ex. 1, 2, 3, 4, 5...), and seeing which digits yield 404 (does not exist) vs 401 (not authorized to access this resource) response codes, they can estimate how many users our application has. Additionally, the digit itself also often is used to encode information about the privelege of the users.[^3] Some applications might use a convention that the highest privelege Admin user has user `id` 1 or 0, and that any user with `id` < 100 has elevated priveleges.

Consider the case where an application has a silo'ed deployment strategy based on geographic location. As a simple example, let us say that our application has a unique deployment per continent. This helps decrease latency by making the application servers closer to the end user. But if each silo'ed deployment is truly independent, then the user `id` is also only unique within that silo! If we ever changed our deployment method, say by moving to a cloud-native architecture, we would have to reconsider a fundamental piece of our database. 

### Mitigations

Obviously, these problems can be solved with some work. For example, you could restrict the entire `/users/` set of endpoints to only respond to logged-in users, and yield a 401 error code otherwise. This would protect against attackers who do not have an account. By restricting the `/users/` endpoints to admin users only, this would further protect against attackers who do have an account.

In our second case of silo'ed deployments, we could have an independent central `users` service which is not silo'ed and serves all of the deployments. This would mean the integer user `id` would be globally unique. But now every request involving users has to be routed to and from a central server, increasing latency and likely leading to dissatisfied customers. 

But when trying to bring an application to market as quickly as possible, selecting a smarter primary key yields a lot of benefits. 

## UUID

The golden standard for user `id` data type is UUID, which is an awesome set of algorithms which allow you to generate universally unique 128-bit numbers. 

[^1]: We are using PostgreSQL in this example
[^2]: `SERIAL` is PostgreSQL shorthand for `INTEGER AUTO INCREMENT`, which tells the system to set up all the boiler plate sequences and sequence relations for you.
[^3]: In linux, user id 0 refers to the root user, who has full admin priveleges