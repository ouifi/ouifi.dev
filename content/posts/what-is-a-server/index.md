---
title: "What is a Server?"
date: 2022-02-20
Toc: true
description: "An exploration of the modern word \"server\". Learn why we use this word, its origins, and why it confuses. Approachable for the layperson but hopefully instills further curiosity in the technical mind."
tags: 
  - tech
  - language
keywords:
  - server
  - cloud
  - virtual machine
  - client
---

# What is a server?

## Intro

My wife, who was a Art History major, loves to ask me technology questions. I enjoy the exercise of trying to [ELI5](www.reddit.com/r/eli5) something to her, and I think it makes me a better technologist. If I can explain a technology to someone with basically no experience, then it only gets easier to explain it to anyone with some experience. 

What is a server? I don't think this question gets asked often enough by junior developers. The word server has become so commonly used in the software development field as to become basically a non-word with no actual meaning. I also want to demystify it. There is nothing about a server which cannot (or should not) be understood by every person working with computers.

Server means different things to different people. To a video gamer, it's the developer-owned machines that are never performant enough for online multiplayer. To the IT System Administrator, it is a building block of the datacenter. 

Through this article I will attempt to provide an in-depth explanation of the word "server". If you are looking for a quick answer, try the summary at the end of the article. But I feel as though this topic deserves a fuller exploration. 

## Why is it called a server?

"Server" as a name arises from a description of functionality rather than a description of form. The first known usage in a context we would maybe recognize comes from David G. Kendall's [1953 paper on queuing theory](https://projecteuclid.org/journals/annals-of-mathematical-statistics/volume-24/issue-3/Stochastic-Processes-Occurring-in-the-Theory-of-Queues-and-their/10.1214/aoms/1177728975.full). We see it used in a more familiar way in [RFC5](https://datatracker.ietf.org/doc/html/rfc5)[^1] as a way to define the server-client relationship over ARPANET. By definition, the server is whatever is on the other side of the connection from a client. 

## The server as a box

{{< figure src="images/physical.png" title="Physical Server" >}}

The usage from RFC5, while abstract, almost certainly was written with the idea that the server and client were physical computers connected over a network. This brings us to the first answer of "what is a server". A server is a physical computer, a box full of silicon and copper that is used to run code which serves some resource over a network. This definition brings to mind images of large datacenter warehouses, owned by Google or Amazon, lined with these gray blinking boxes. 

These physical servers are not all that different from the computer with which you are reading this article. There is nothing so foreign or alien about them. They are computers with a CPU, RAM, a motherboard, network card, and power supply. Most likely, they are running an Operating System (OS) different from yours. They are optimized for a different environment and usage, but they are still just computers. 

You don't drive a Formula 1 car to work every day. It would be a way more powerful, way less efficient way for you to commute. Its impractical for everyone to have a Formula 1 racer. But for the very specific purpose of being a high performance machine that can run at a million miles an hour, you cant just grab a Toyota Camry off the used car lot. 

This hardware-centric definition is the most useful one. From here, we have context for definitions that came later with additional technological advancement.

## The server as a software

{{< figure src="images/software1.png" title="Software Server" >}}

Now let us try the full other end of the spectrum.

Recall earlier that even in the inherently hardware-oriented definition of a server, there was still a caveat that the server was running software to serve clients. These softwares are also commonly referred to as servers. You may have heard of an HTTP server. This is a piece of software which can communicate over a network (serve) with a client and exchange data using the HTTP protocol. 

Any active HTTP server software, no matter the hardware it is running on, could be accurately identified as a server. 

If you want to try this yourself and you have python installed on your computer, run this in your command line of choice. 

```bash
python3 -m http.server 8001
```

Then open your browser to point at `http://localhost:8001`. Your computer is a server! ... and the client. 

This looks like the next figure.

{{< figure src="images/software2.png" title="Software Server" >}}

Software servers comprise many common software we use every day. Email, Files, Video Game, Printing, Media, Database, Sound, Proxy, Communicatons. These are all examples of Software Servers. 

Using our car analogy, these software servers are the engine. Without the engine, the physical server couldn't do anything and would probably risk losing the title of server. 

Developers working on server software fall into this exact category. They usually run the client software and the server software on their own machine. This means they can iterate more quickly. 

## The server as hardware & software?

{{< figure src="images/hardware_software.png" title="Software Server" >}}

Living somewhere in the middle of hardware and software are Virtual Servers. Also called virtual machines, these are fully realized "virtual" servers that are emulating physical servers through code[^2]. The virtual server can have virtual USB ports, a virtual monitor, a virtual CPU.

If you have ever gotten too into Minecraft, you might have looked into renting a Virtual Private Server (VPS) through a service like DigitalOcean. If you have used cloud services, Amazon EC2 and Google Compute Engine are also used to create virtual machines. 

Virtual Machines, being fully featured operating systems, have all the features of physical server boxes, while having the advantage of being software defined. For example, to increase the size and capability of a virtual server, you can simply modify the settings[^3]. 

Effectively implemented, it opaque be completely opaque to the end user what kind of server they are accessing. I would say that this use of "server" is the most commonly intended definition in modern tech parlance. 

Most often nowadays, physical servers exist only to host multiple virtual machines. It is only in exceptional cases where dedicated or specialized hardware[^4] is needed to run a program that such a program is run directly on a physical server. 

Back to our car analogy, this is like if a Formula 1 racer was powered by 10 tiny Toyota Camry's trapped in the engine compartment...

Again, just to reiterate, there is nothing particularly "server-ful" about running virtual machines. You could just as easily run a bunch of virtual machines on a "client" computer, running client software and call them "clients". It is simply that the use case for a whole bunch of servers is more common than a whole bunch of clients. 

## Serverception

The final point to make is to take this extension of servers to its natural conclusion. As mentioned earlier: well executed, a virtual server should be completely opaque to an end user. This means you can indeed run virtual machines inside of virtual machines, treating the host virtual machine as if it was a physical server. 

To demonstrate, recall that both Amazon and Google offer VPS's for rent. Say, for example, that I am a user that would like to run some virtual machines. Maybe I want to host both a Minecraft and a Terraria server, but I want them completely isolated from each other, so that if my Minecraft server gets really popular, it wont affect my Terraria server because each virtual machine has its usage limited by my software configuration. 

This brings us to our ultimate diagram, which I affectionately refer to as "server-ception". Its servers, running on servers, running on servers, running on a server, serving clients.

{{< figure src="images/serverception.png" title="Software Server" >}}

In practice, more than 1 level of nested virtual machine is not really used[^5]. Each successive level of emulation incurs a performance penalty. There are other methods of virtualization which get around this restriction, though.

If a Formula 1 racer really was powered by 10 Toyota Camry's trapped in the engine compartment, it wouldnt be a very good racer. Especially if each of those Toyota Camry's was actually powered by 10 even tinier smart cars trapped in their engine compartments. 

## Conclusion

So now we have come full circle. It is clear that the question "what is a server?" is not so trivial at all. When my wife asked me this question the first time, I probably would have just given the hardware-centric answer. But as I went down the rabbit hole of system administration and IT, my answer changed every time she asked. 

I hope this article has demystified servers for you, but I understand if you stopped reading halfway through. If you take one thing away from this piece, take away this. A server is NOT fundamentally any different from your computer. It is a piece of specialized equipment, running specialized software. But so is your PC, your iPhone, and your smart refrigerator! 

[^1]: Jeez remember single-digit RFCs?
[^2]: The word "virtual" in computer science almost always means a physical thing being emulated by software. 
[^3]: The word "hypervisor" is a piece of software which manages virtual machines on a host.
[^4]: Such a case might be high levels of privacy, so the software legally cannot be run along side any other software on the machine. Another case might be the need for high-performance computing, where dedicated physical components with no software emulation justifies the high cost of purchasing physical hardware.
[^5]: Additional layers of virtualization are usally realized using containerization technology such as Kubernetes or Docker. These virtual systems are more lightweight than full virtual machines. 