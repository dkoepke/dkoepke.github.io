---
layout: post-no-feature
title: "A Paean to the Monoliths"
description: "There are no guarantees conferred by your architecture that cannot be undone by your implementation."
category: articles
tags: [distributed systems, architecture, service-oriented architecutre, SOA, micro-services, scalability, monolithic, software engineering]
---

Perhaps because they are so poorly understood, there is quite a lot of hype about service-oriented architectures (SOA) and micro-services. It is not that all of this hype is unjustified. When it comes to building beautiful software that meets all of its functional and business requirements, services can be a great tool. But all tools are ineffective when misunderstood, and the advantages and disadvantages of SOA are badly misunderstood.

Consider this criticism of non-service-oriented architecture in [Firat Atagun's Analysis of Software Architectures](http://www.firatatagun.com/blog/2016/01/09/analysis-of-software-architectures/), "While layered applications can perform well, since requests [have] to go through multiple layers, [they] might have performance problems."

Yet no mention is madee that distributed systems are more vulnerable to this critique. Service-oriented architectures have more layers of greater complexity. The network, the marshalling layer, and the service bus (or queue) are all potential performance bottlenecks that require specialized knowledge to tune.

A common mistake is to not even consider the performance impact of durability and availability of the service bus on your application. Once your queue must confirm writes, your throughput can crater, spreading a wildfire of failure through your system. The issue of what to do when components become unavailable or unhealthy is a huge concern when doing service-oriented architectures right. In a monolithic architecture, the application as a whole is available or not. In SOA, services can come and go. If you think the answer is that the components are independent, so you don't have to think about it, you're in for unpleasant surprises.

### The illusion of isolation

One of the touted benefits of SOA is isolation, but total isolation is illusory. Services are isolated only in the sense that the code and the container running it are isolated. But the running services are certainly not isolated from each other. Services depend on each other or, at least, on the message bus or the network or the cluster hosting the containers. This creates subtle dependencies that can be the source of many problems. When you don't think about services' side effects and how it can all go awry, you don't do the hard and interesting and necessary work of architecture.

What happens if one of your services gets publish-happy? Will contention prevent other services from having their work handled? Fair queueing of reads is not a complete answer since a talkative service may consume the write capacity, delaying or preventing other services from publishing. *There are a million little edge cases that crop up as your architecture accrues moving pieces.*

### The false appeal of simplicity

So many analyses of software architecture ultimately fail because they ignore too much of the complexity of distributed architectures to ever get to their real advantages. Architects tend to overstate the scalability, availability, and simplicity benefits of service-oriented architectures. *Service-oriented architectures perform worse, are less reliable, and are harder to scale than a comparably built monolithic application.*

We all have horror stories of slow and crashy monolithic apps, but it's not as if the people that write terrible monolithic apps are going to start writing great services. It's not the architecture that's to blame. It's the people writing the code and the processes that allow technical debt to mount and monoliths to crumble to dust. The people that write hard-to-scale monoliths will write hard-to-scale micro-services because they haven't thought about or don't understand what trade-offs to make for scalability.

All software engineering involves the management of trade-offs, but there are more to make in a service-oriented architecture. This flexibility is not a bad thing, but it is a complication. Complexity is not bad in and of itself. But unnecessary complexity is a great evil. Service-oriented architecture is more complex than monolithic architecture, so you must understand the real benefits you are getting and at what cost.

### Scaling teams, not software

*The real scalability benefit of a service-oriented architecture is that you can scale your development teams*. For a while, your team's productivity increases with head count. But as a team grows, you need heavier processes, and your returns on productivity eventually start to decrease. There is an inflection point. At a certain size, the team's productivity will decrease with each new person.

This is an inescapable reality. It's the big knock on enterprise culture, where employees fill out useless forms and go to endless meetings where nothing is accomplished. Those stupid forms and pointless meetings are necessitated by the fact that large teams cannot be directly managed, so managers distribute part of the management tasks to workers. And part of a manager's job is to deal with the dumb bullshit that happens in any human organizaiton.

The solution for all of this is to have more and smaller teams that can work independent of each other. Services allow this because they are separate and smaller products that can be independently defined, implemented, maintained, and shipped. But under a certain team size, you don't need this benefit, and when it comes to scaling your application to support more users, service-oriented architectures are only a form of cost engineering.

Monolithic software tends to require beefier hardware, but it's not necessarily impossible or even hard to horizontally scale monoliths. The real barrier to scalability is statefulness, and there's nothing inherently stateful to monoliths. More troublesome, there is nothing inherently stateless about services. Plenty of people are building micro-service architectures that don't scale well. You still need to think about scalability and design for it. Smaller services tend to have cheaper hardware requirements, making them cheaper to scale, but don't jump to services thinking they'll **make** you scale. There are more ways to scale an application that is composed of services, but that means there's a lot more thinking to do about how to achieve your scalability goals. Figure out how to scale, then figure out if services will help you get there on your budget and without undue complications.

### All for one, and one for all

Similarly, you should never confuse the fact that service-oriented architectures give you more deployment options for the idea that services are easier to deploy. There are real benefits in having more deployment options, and some of those options might be simpler than the options available to a more monolithic application. But there are no guarantees conferred by your architectural choices that cannot be undone by your implementation. Services do not grant you zero downtime deploys. They do not grant you the ability to upgrade one piece without impacting other services. *You don't win a solid deployment strategy by picking the right architecture. You build it.*

I've seen this failure in real life. A data collector reports to two services which house different types of data and are completely independent of each other. But if you plotted the incoming data rates against the uptimes, you would see that when one service is down for maintenance or upgrade, the other stops writing data. When this was first observed, there was disbelief: the services are independent and do not talk to each other at all. They aren't even aware of each other. But the collector talks to both services, and the collector is configured by one of the services. If that service carrying the configuration is down, the collector cannot send well-formed data to the other service. A lot of effort went into making these services technically independent of each other, only to end up at a place where they are inextricably entwined. It should've been seen from the very beginning, but the promise of services can blind you.

One of the reasons this glaring problem was missed is the flip-side of how services help you scale your teams. Having smaller teams that are more focused on delivering an individual service is great, but it increases the likelihood of your organization missing out on the big picture. You want your teams to **be** independent but not to **act** independently. When teams are dependent on each other, there are blockers that slow progress and create resentment. When teams act independently, they fail to communicate and don't deliver on the actual goals.

### Breakin' 2: Electric Bugaloo

Failures in team communication lead to systemic failure, which is a frightening parallel to how failures in one service can rapidly spread to others. In my real world example, it was simply that one service's unavailability made both services unavailable because the dependency between the services was encoded outside of their boundaries. But it's also possible for service outages to spread through a system even when every component is available, or even when every component, considered in isolation, is behaving as it should.

Let's say one service is dealing with high load and so is running a few milliseconds slower than usual but still within our tolerances. Our next service down the line can adopt and magnify that latency--even in a fire-and-forget, asynchronous scenario--on connection. It doesn't take much to turn sub-optimal performance into timeouts, those timeouts into retries, and those retries into something that deepens and prolongs an outage. We need to define operational constraints on our interfaces to make services fallback (and eventually fail) gracefully when their dependencies start to become unhealthy. Architecture analyses that don't mention circuit breakers are of no use. Architects that don't understand this aren't worth their titles.

The interaction between different services is surprisingly complex and mostly unspecified. I've seen many a service defined strictly in terms of its interfaces and wire protocols: it speaks JSON over HTTP, and here's the schema for the payloads. The better interface definitions include some semblance of error handling. The best ones deal with backing-off and falling back when failures are high. Yet, even if you specify what clients ought to do in certain exceptional cases, the actual behavior is not isolated to the affected service: it's the clients responsibility.

Engineers make the mistake of treating service calls as if they were function calls. The only time you get away with this category error is if having an SOA versus a monolithic architecture never really mattered for your operational needs.

### The right tool for the job

Due to code-level isolation, SOA makes it possible to implement services using a variety of different languages and frameworks. Whether this is an advantage or not really depends on how well your teams coordinate and communicate. The operational cost of using and deploying different technologies and of understanding how to tune them cannot be ignored.

If your teams are choosing the tool that accomplishes the goals with the least amount of development effort and the lowest operational and maintenance overheads, then the freedom of choice is a benefit. There still needs to be intention and coordination behind the decision.

Everyone knows that it's foolish to choose a technology just because it's trendy. But it's also a mistake to choose a tool just because everyone on the service team is an expert in it. *The entire organization needs to be thinking about the total cost of ownership*. Expert employees come and go, so you must consider the cost of hiring a replacement for a service written in an uncommon platform. And when a team needs more help from within the organization, different technology choices can establish a barrier at a critical juncture.

This is why freedom of implementation technology is only a clear cut benefit for larger organizations. Smaller organizations can (and often should) standardize on a handful of tools that are good enough and that they know how to hire for. Since the organization is small, it's easy to migrate people to new tools. Very large organizations move at a glacial pace, if they're not completely stuck in the mud, so some degree of freedom is a positive. Even then, too much freedom descends into anarchy and chaos.

### The rewrite tool for the job

An indisputable advantage of SOA is that small software is easier to rewrite, and services tend to be smaller. Rewrites are a kind of evil, but evil is sometimes necessary. When a well-defined service fails to meet its functional, operational, or maintenance requirements, or when it accrues too much technical debt, it is ripe for replacement. Micro-services are prime targets for clean reimplementations.

Be careful when you tout this goal. Rewriting services is a form of organizational waste. You are not advancing the business goals by rewriting software. At best, you are removing blockers in a very inefficient manner. Many times, the rewrite is not to achieve any actual goals: it's just hard to understand existing systems that other people wrote, and easy to think you can do better.

*You need to seriously consider whether and why the disposability of services is useful and attractive to your organization*. It might be of most interest simply because you are bad at keeping forward momentum on the important pieces. Since services are (ideally) small, well-defined, and isolated, it's easy to fool yourself into thinking that rewriting them is a low-cost solution to some problem. This is like those people that argue a sale saved them money. If you weren't going to buy it at $200, but do buy it at price $150, have you saved $50? No, all you've done is spend $150.

It is up to you and your teams to make the disposability of services a net positive instead of a source of continual waste and problems. As with the freedom of implementation choice, this is a benefit mostly to larger and more successful organizations. Very large organizations can better endure the cost of a rewrite and the risk of a rewrite failing.

### Increased coordination has a cost

There's an underlying theme in all of this. Architects sometimes paint service-oriented architectures as making various parts of life simpler and better. But all engineering is trade-offs. If one thing is easier, something else is harder. The real story of SOA is: you have more choices. More choices requires more thought and coordination and understanding. The devil is in the orchestration of services, the work of defining fallback and failover behavior, the communication between independent teams, the forwards and backwards compatibility contracts, the wire protocols that leverage your network design, the choice of implementation technologies for long-term maintainability, the decision to rewrite or patch.

The industry at large needs to understand this better in order to build better software: *every architecture has its costs, but you can't pay them if you don't acknowledge there's a bill*. So maybe you shouldn't build a monolithic application in 2016, but what you absolutely must do is build systems that you and your team thoroughly understand. Even in 2016, those systems might look more like a monolith than micro-services.


#### Change log

* 10-01-2016: Added sections "The right tool for the job" and "The rewrite tool for the job" to discuss some benefits large organizations see with services.
