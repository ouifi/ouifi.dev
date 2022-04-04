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
> Alright, I have got my Git repos, let me start writing API endpoints. First I will write a `/users/` set of endpoints.Most applications have users. And now to write the `user` table in the database. Ok, easy; let's start with `id`, `name`, `email`. Nice! Making great progress. Let's pick our data types. `name` has gotta be a string, so let's go with `TEXT`.[^1]. `email` should be `TEXT` too, and we will enforce the format in the API, no problem. `id` should probably just be an integer, starting with 1. And make it the primary key too! Then we can reference it in other tables. Hm, before I get too deep, let me just Google this to see what other people think. 

And that is how I found 10 articles with 10 opinions about selecting a primary key.

## What's wrong with integer?

This was my first question. Almost every other time I have written database/SQL code in my (short) career, I default to having an integer `id` field in every table. This forces the table to conform to 3rd normal form in most practical cases. 

[^1]: We are using PostgreSQL in this example