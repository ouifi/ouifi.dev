---
title: "What is a Server?"
date: 2022-02-20
draft: true
Toc: true
description: "An exploration of the modern word \"server\". Learn why we use this word, its origins, and why it confuses. Approachable for the layperson but hopefully instills further curiosity in the technical mind."
tags: 
  - tech
  - language
---

{{% draft_message %}}

# What is a server?

## Intro
My wife, who was a Art History major, loves to ask me technology questions all the time. I enjoy the exercise of trying to [ELI5](www.reddit.com/r/eli5) something to her, and I think it makes me a better technologist. If I can explain a technology to someone with basically no experience, then it only gets easier to explain it to anyone with some experience. 

What is a server? I don't think this question gets asked often enough by junior developers. The word server has become so commonly used in the software development field as to become basically a non-word with no actual meaning. I also want to demystify it. There is nothing about a server which cannot (or should not) be understood by every person working with computers.

Server means different things to different people. To a video gamer, its the developer-owned machines that are never performant enough for online multiplayer. To the IT System Administrator, it is a building block of the datacenter. 

Through this article I will attempt to provide an in-depth explanation of the word "server". If you are looking for a quick answer, try the summary at the end of the article. But I feel as though this topic deserves a fuller exploration. 

## Why is it called a server?
"Server" as a name arises from a description of functionality rather than a description of form. The first known usage in a context we would maybe recognize comes from David G. Kendall's [1953 paper on queuing theory](https://projecteuclid.org/journals/annals-of-mathematical-statistics/volume-24/issue-3/Stochastic-Processes-Occurring-in-the-Theory-of-Queues-and-their/10.1214/aoms/1177728975.full). We see it used in a more familiar way in [RFC5](https://datatracker.ietf.org/doc/html/rfc5) (jeez remember single-digit RFCs?) as a way to define the server-client relationship over ARPANET. By definition, the server is whatever is on the other side of the connection from a client. 

## The server as a box

{{< figure src="images/physical.png" title="Physical Server" >}}

The usage from RFC5, while abstract, almost certainly was written with the idea that the server and client were physical computers connected over a network. This brings us to the first answer of "what is a server". A server is a physical computer, a box full of silicon and copper that is used to run code which serves some resource over a network. This definition brings to mind images of large datacenter warehouses, owned by Google or Amazon, lined with these gray blinking boxes. 

These physical servers are not all that different from the computer with which you are reading this article. There is nothing so foreign or alien about them. They are computers with a CPU, RAM, a motherboard, network card, and power supply. Most likely, they are running an Operating System (OS) different from yours. They are optimized for a different environment and usage, but they are still just computers. 

This hardware-centric definition is the most useful one. From here, we have context for definitions that came later with additional technological advancement. 

## The server as a software
Now let us try the full other end of the spectrum. I will try to keep this section approachable for a broad audience, but I may dip into software terminology unfamiliar to a non-technical reader.

Recall earlier that even in the inherently hardware-oriented usage of a server, there was still a caveat that the server was running software to serve clients. These softwares are also commonly referred to as servers. You may have heard of an HTTP server. This is a piece of software which can communicate over a network (serve) with a client and exchange data using the HTTP protocol. 

Any active HTTP server software, no matter the hardware it is running on, could be accurately identified as a server. 

If you want to try this yourself and you have python installed on your computer, run this in your command line of choice. 

```bash
python3 -m http.server 8000
```

Then open your browser to point at `http://localhost:8000`. Your computer is a server! ... and the client. 

Software servers comprise many common software we use every day. Email, Files, Video Game, Printing, Media, Database, Sound, Proxy, Communicatons. These are all examples of Software Servers. 

## The server as hardware & software?

Living somewhere in the middle of hardware and software are Virtual Servers. Also called virtual machines, these are fully realized "virtual" servers that are emulating physical servers through code[^1]. If you have ever gotten too into Minecraft, you might have looked into renting a Virtual Private Server (VPS) through a service like DigitalOcean. If you have used cloud services, Amazon EC2 and Google Compute Engine are also used to create virtual machines. 

Virtual Machines, being fully featured operating systems, have all the features of physical server boxes, while having the advantage of being software defined. For example, to increase the size and capability of a virtual server, you can simply modify the settings[^2]. 

Effectively implemented, it should be completely transparent to the end user what kind of server they are accessing. This is a personal judgment, but I would say that this definition of "server" is the most accurate one in modern parlance. 




[^1]: The word "virtual" in computer science almost always means a physical thing being emulated by software. 
[^2]: The word "hypervisor" is a piece of software which manages virtual machines on a host.