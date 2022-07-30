---
title: "Don't Ignore Git Ignore"
date: "2022-07-30"
publishDate: "2022-08-15"
Toc: true
tags:
  - git
  - tech
keywords: 
  - git
  - gitignore
description: "A look at all the ways to gitignore"
---

# Don't Ignore Git Ignore

## Intro

Most developers don't think about git ignore in their daily life. The `.gitignore` file is something you create at the start of your project and maybe adjust once in a blue moon. The `.gitignore` system works well, and other systems have adopted the same approach.[^1]. 

## The Git Ignore File

The advantage of the `.gitignore` file is that it is version controlled and lives along side the source code it protects. Often it includes the files (or patterns of files) of the build artifacts, test artifacts, etc. 

Don't forget, that you can have multiple `.gitignore` files. Most repositories only utilize the project root-level one, but the rules are only that `.gitignore` files affect sibling and children directories. So if you have a more complex project, consider adding `.gitignore` files with more specific rules in the directories where they are needed. Keeping them all in the top level `.gitignore` file could (rarely) lead to a conflict. You could also use a subdirectory `.gitignore` to disable rules from the top level `.gitignore` file. 

What most developers don't know, is that the `.gitignore` file is only one of a few ways to actually get Git to ignore your files. 

## Project Level Git Ignore

What if you have a file/folder pattern you want Git to ignore, but you don't want to clutter the project `.gitignore` file? One use case for this that I often run into: I want to track information related to a project, but that does not belong in the source code. For example, when I make some code edits and need a fix broken unit tests, I will put the list of unit tests into a text file. I could just make sure not to commit these files, but this is additional, unneccessary mental burden.[^2] If I mess up and commit/push these files, their contents are forever entrenched in the git history, or I have to go through the laborious process of undoing/redoing the commits since then. 

A better way to solve this is the Git Info Exclude file. Every Git repository has a hidden `.git` directory. In this folder, Git stores all the info it needs to do the git things. For example, the location of your checked out location is stored in the `.git/HEAD` file. You can just go look at it!

In the file `.git/info/exclude` file, you can use the same `gitignore` patterns. So for my use case that I outlined above, I might add the content

```bash
test-failures*.txt
```

## User Level Git Ignore

Every user in a *Nix type system should have a Git config file located at `~/.gitconfig`. In this file, you can add the following contents to enforce `gitignore` rules across all projects you work on!

```toml
[core]
    excludesFile=~/.gitignore
```

Then, you can create your user-wide `~/.gitignore` file. The question is, what to put in it? The Git handbook suggests putting file/folder patterns there for content created by a user's text editor of choice. In my case, it would make sense to add a line with `.vscode`. 

My personal favorite line to add there (especially if I am working on a Mac) is `.DS_Store`. Oh the bliss! To never worry about polluting my fellow developer's machines with this heinous file is true ascension. If you are a Mac developer, I plea with you to add this to your user-level `.gitignore` file. Your fellow devs will thank you. 

## Conclusion

Today, we learned about many of the ways you can configure Git to ignore certain files. I want you to click away from this article knowing that you can use `gitignore` smarter! This article definitely is not my hate-letter to the `.DS_Store` file. Nope. 

In all seriousness, it is probably still good to include `.DS_Store` in your repository's `.gitignore` file.[^3] But when its included in your user-level config, you (and your coworkers) are now forever freed from seeing it pollute their precious source code. 

I encourage you also to play around with these rules. Is there a dev workflow you employ that would be improved by configuring something in the `.git/info/exclude` file? Obviously you can use `git status` to check if Git notices file changes in a repo, but you can also use `git ls` to see what files Git even considers.

[^1]: https://docs.docker.com/engine/reference/builder/#dockerignore-file
[^2]: What developer wants to do anything other than `git add .; git commit` anyway?
[^3]: Most default `.gitignore` suggested lists worth their salt include this anyways