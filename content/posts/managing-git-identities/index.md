---
title: "Managing Multiple Git Identities"
date: "2022-03-15"
publishDate: "2022-03-15"
tags:
  - tech
  - git
  - ssh
Toc: true
description: "A guide on how to manage multiple git identities."
---

# Managing Multiple Git Identities

## Introduction

Problem Statement:
> I have multiple remote git providers that I am pulling/pushing code from, with different accounts on each. How can I automatically use the right account for each project?

The solution to this problem depends on effectively utilizing the main technology that git relies on for communication: SSH. 

1. Have 1 public/private key pair per identity
2. Configure SSH to use the correct identity for each remote git provider

## A Layperson's Explanation

Batman and Bruce Wayne are the same person, we know that. But the public does not know that. Bruce Wayne is allowed access to board meetings and charity events, while Batman has access to...anywhere he wants. This is a single person with multiple identities. There would be at the very least awkwardness if Batman attempted to attend a board meeting and Bruce Wayne walked into the Joker's hide out. He has to use the right *identity* for the right place. 

So, we just need to make sure you have the cowl and batsuit when you need to be Batman, and a tuxedo with coat tails when you need to be Bruce Wayne.

## The simplest case

This is the starting point for most people. They use git over ssh, and know to add their public key to the remote git provider whenever they want to access repositories securely. 

{{< figure src="images/simple.png" title="One git identity, multiple remotes" >}}

For most people, this is a fine default. Each remote git provider (GitHub, GitLab, BitBucket, etc.) has a copy of your public key, and can verify your identity when you push/pull. You can just keep adding git providers and this will work ok!

## +1 Complexity

You have been programming for a while. You do your work development on your work computer and you do any personal development on your personal computer. One day you want to do a little programming on a personal project on your lunch break. You sign in to your personal github account and go to add your owork computer's public key. You get an error message! This public key has already been associated with another account. 

This makes sense. If GitHub allowed you to do this, it would end up having no idea which account you mean to use when you push/pull code. When you attempt to interact with a git provider over ssh, your public key is used to verify your identity. 

Then, to interact with the same git remote provider using multiple accounts, you just need to assume another identity!

Our goal looks something like this:

{{< figure src="images/complex.png" title="Multiple git identities, multiple remotes" >}}

## The Solution

Check the contents of your `~/.ssh` directory. If you see `id_rsa` and `id_rsa.pub`, then you already have your Bruce Wayne outfit ready to do. If not, perform the following actions twice, once for your Bruce Wayne identity, once for your Batman identity.

Create a public/private key paid, using the RSA algorithm, with 4096 bits, and save it as batman.

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/batman
```

No need to enter a passphrase[^1] if prompted.

Now, we have established multiple identities. So the next critical step is to tell ssh when to use which identity. 

Create the ssh config file:

```bash
touch ~/.ssh/config
```

And use this example as a guide to populate it

```
Host batcave
    Hostname github.com
    User git
    IdentityFile ~/.ssh/batman

Host wayne-manor
    Hostname github.com
    User git
    IdentityFile ~/.ssh/bruce_wayne
```

Finally, we need to ensure that our git repositories know which `Host` to reference. The ssh URL for cloning a git repository looks like this:

```
git@github.com:facebook/react.git
   |----------|
     the host
```

To note that this is a repository that we should access using the batman identity, use the batcave host.

```
git@batcave:facebook/react.git
```

And ssh will take care of the rest!

To see the URL of an existing git repository's origin remote, use

```bash
git remote get-url origin
```

To change the URL, use

```bash
git remote set-url origin git@batcave:facebook/react.git
```

## Conclusion

Today, we learned how the SSH tool manages our identity and how that impacts our git usage. We also learned how to use SSH to make it easier to switch between different identities on the same git remote provider. Because we are using SSH without any complex abstractions, this exact same strategy could be used to manage ssh-ing into many different servers! I will leave implementing this as an exercise for the reader.

[^1]: Unless you are on a system with many other shared users and you think someoe may steal the private key to impersonate you. 