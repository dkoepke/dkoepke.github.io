---
title: "Going Gold"
description: "Shipping better software, sooner"
category: articles
tags: [sdlc, software engineering, general availability, mvp, release planning, milestones]
---

In [reflecting on Continuous Delivery](/articles/tech-execs-overheard), I noted that engineers have an unfortunate tendency to treat each release as if it was our baby: painful to birth and the source of sleepless nights for months after. This problem is magnified for the first ever release: the much dreaded GA (general availability) release. This is the release that's been incubating for months, pure and unsullied by the wanton hands and demands of customers, and you are being asked to hand it over to the wolves, and you know it's not ready for the big, bad world yet.

In my career, I've had the opportunity to take about ten new products to GA. In some cases, the product began with zero lines of code and no team: a complete greenfield. In others, the product was languishing in an indeterminate state, with a lot of work done and desired release dates but an incomplete vision of what the released product would be and no path to get there. Many a product has just needed that *one last push* to get to the finish line, only to discover that we didn't actually have a clear, shared idea of where the finish line was.

Taking a product to GA takes a lot of work, but it doesn't have to be messy and painful. I learned a lot of the process behind it by screwing it all up, getting behind, breaking down, and having to drag myself out of the hole I dug with herculean (or perhaps back-breaking) efforts. I want to share a few of these lessons with you, in the hopes that you'll find going GA a relief instead of a nightmare.

### Challenges and Opportunities

Regardless of the current state of the product, whether it's greenfield or almost there or off-track, you will face three main challenges.

*How do we define what goes into GA?* We are working on this product because we believe it has potential, and we (hopefully) have prospective customers in mind that have ideas about what this product should do and how it should do it. We want do to everything. The product could do everything. Customers want everything. But we can't do everything. When I started out the Python agent, there was a lot of talk about the need to support ancient and new versions of Python, async, ten different frameworks, CLI and web and batch jobs, etc. It's great to hear all of the myriad things your product could do that would be valuable, but it can be overwhelming and leave you with no clear picture of what you really need to do.

Still, there's a lot to do, so *how do we go fast but keep quality high?* It's much harder to pave the road for success when you're already driving full speed on it, so you need to consider resourcing and quality an essential part of the product requiring as much time, thought, and energy as the features.

Finally, *what does releasing a product actually look like?* These are all of the minutiae of building and packaging and releasing the product. It's the bow that ties everything together.

The bad news about these questions is that there is no single set of answers. Each product and each situation is different. But there is a framework, and it starts with something you've heard of.

### Minimum Viable Product

This concept is so discussed and dissected that it is hardly worth further discussion. Your professors told you about it in school. Your colleagues all talk about MVPs. The CEO for <span style="font-style:italic;">that one startup</span> spent his whole time interviewing you talking about their MVP and product-market fit, and you still count your blessings for declining that offer and dodging that bullet.

In short, the MVP is clich&eacute;. But it is clich&eacute; for good reason, and if there's any color I can add to this monochromatic idea, it's that each of these words carries a lesson that you need to learn. I call these lessons the way people "cheat" on the MVP.

People cheat on **minimum** by just doing what customers or their bosses or their imaginations say. You need a coherent vision--a narrative--for your product, especially when it comes to version 1. Cherry picking the features you think you can comfortably put in before the deadline isn't being minimal. Each of the features you pick must play a part in the story you're telling. You could end up with a successful release with a random assortment of features, but you will certainly find that some of those features weren't actually necessary. You actually built beyond the minimum viable product. To avoid that, listen more to what your customers are **telling** you than to what they are saying.

People cheat on **viable** by pushing out the hard work. If part of your core value proposition is god damned difficult, don't push it out to version 2. That doesn't mean you have to ship it in version 1, but if your product is ultimately a dead-end without that feature, you better have done your feasibility and design and figured out if this is really something you want to build. I've seen projects that had to be torn down and restarted because the team delayed the hard work that was central to the product's long-term success to deliver the low-hanging fruit immediately. Easy wins are only wins if they're along the path you'll take.

People cheat on **product** by simply ignoring its existence and implications. You aren't building a codebase, framework, library, service, project, database, or application. You aren't writing code. You are building a product. That means you need to consider the entire experience you're building, from why someone would want it, to how they will install it, to how they will use it, and what happens when it goes wrong. The code is only one piece of the picture. Engineers don't just get to think about the algorithms. We need to listen to, support, and augment the product people, so that we are perfectly aligned with the product vision.

So let's say you aren't going to cheat on the MVP. What now?

### Leverage Existing Work

The best feature is the feature you get for free. When building [the Python agent for AppDynamics](https://www.appdynamics.com/python), I was careful to consider the fact that our existing Application Performance Monitoring platform gave me some features for free. Just by connecting to the platform, Python tiers would show up in the flow map, and the nodes would appear in the inventory of machines.

That's a small thing that involved the bare minimum of work for the agent--the agent didn't even do much of anything except start up. But I've seen first hand how important this feature is. I've worked in and with enterprises that had no visibility into their own infrastructure. Many years ago, I worked at a large enterprise with my friend Alex, who these days works at Stanford, and I will never forget the flyer he Photoshopped up because there was a problematic server in the data center that nobody could actually find. I've also seen enterprises where critical services were discovered to actually be hosted off someone's desktop. Knowing all this, I wasn't afraid to treat the platform features I got "for free" as a key part of the product I was launching. Cheap wins often pay for themselves.

### Paving the Road to Success

There are things you know you will need. You need good automated testing so you can refactor and catch regressions before they ship. You need documentation. You need a way of evaluating whether your product is meeting customer demands. You need build, release, and upgrade machinery. You need good logging and debugging tools for customer support for when something goes wrong.

Investing too early in features, optimization, and architecture is a risk not worth taking. But you know you need to be able to test your software, you know you will need to refactor it (and sooner rather than later, as I will discuss below), you know people will need to have instructions on what to do, and you know issues will arrive in the field. So invest in this stuff. Don't over-do it at expense of the product's vision, but don't ignore it or leave it for later. You want to always be ready for release. Ask yourself, "What if we suddenly decided the current feature set was good enough for a release? How soon could I release what's currently on master?" If it would take more than a couple of days, you haven't invested enough. This also points to ensuring that your builds are always green on master: use feature flags and get your coverage high and keep it there.

### Drive to Demo

Define success metrics for your product and continuously evaluate whether you are on the road to success. To do so, get the product into a demoable form as soon as possible and start collecting feedback. This has the desireable side effect of focusing on delivering value instead of building architecture. You want to always have "just enough architecture" to do what you need to and no more. Treat tech debt like a credit card: be mindful of your limit, spend some, then pay off the debt when it comes due. To steal a quip from a coworker, you shouldn't need a Black AmEx to cover your tech debt, but you can afford to spend some.

Your job as an engineer is not to write the most perfect code, implement the best algorithm, design the finest class hierarchy, or make an extensible core with dynamically loaded modules. You are building a product that will deliver real value. You will never demo how flexible the underlying classes and architecture of your product-to-be is. Like MVP, YAGNI is a clich&eacute; with power.

In addition to demoing your features, demo **quality**. Show off your test suite to your coworkers. Be proud of the fact that the product works and that you've put in place the tools to allow you to refactor to add more architecture as needed in the future.

You can, and should, also demo the problems, misfeatures, and uncertainties of your product, even to external customers. Demoing is a compromise: your product is not yet in a state where anyone else will use it, but you want the feedback that you would get if they were using it. When people excessively stage manage a demo, they rob themselves of the essential feedback that makes the demo even worth doing. Show the rough parts and talk about how you plan on fixing them. It's fine. People will understand.

### Validate and Revalidate

As you move along, use your demos, the feedback you get from them, and the quality and success metrics you defined to validate your vision. Use everyone you have at your disposal: product management, customer support, technical writers, other engineers, operations people, prospective customers. Do not forget to consider whether there are internal prospects that might use your product (a process rather queasily referred to as "dog fooding"): the first production deployment of the Python agent was for various parts of [appdynamics.com](https://appdynamics.com).

### Jump to Beta

The leap from having a demoable product to a beta is made the moment that someone says they'll use the product you just demoed. Since you've shown the warts, the customers won't be surprised by unforeseen issues. Since you're always ready for release, there isn't a month of work to do to get something releasable. Click the button to cut a beta package, get the customer to sign a beta agreement, and send it over. Since you've invested in the documentation and support stories, and worked with those teams to validate the product at every step of the way, there's nothing holding you back. Once someone is willing to use the product, get it to them.

Plan on the first couple of betas having a very fast upgrade rate. You need your customer ready and willing to do upgrades once or twice a week for the first few weeks. No matter how good your testing, no matter how thorough your vision, no matter how much you think you understand your customer, the first external person to use your product will do something you never imagined anyone would do, and your software will break in some unexpected way in response to this inexplicable action. Be mindful that this issue--of getting your customer to upgrade frequently--is not solved by continuous release. It is, of course, extremely useful to have continuous release in place from the beginning, but even with it there, you need buy-in from your beta customers that they can and will help out by installing new releases at a high enough pace.

Establish a cadence that sees the number of issues steadily diminshing. Focus the dev cycles after the beta release on stability. As you do this, you are asymptotically approaching GA.

### GA Ain't Nothing Special

But when then do you go GA? The answer, somewhat surprisingly, is that if you've done all the hard work at this point, it doesn't really matter. GA is just a label, and that label is often decided for operational reasons instead of for anything intrinsic to the product. If you have a beta that has stabilized; you see demand for the product you've built; and you have buy-in for the marketing, documentation, and support stories, then you are ready for GA. More than once, I've been involved with betas where customers were literally clamoring to buy the product **as is**, if we just called it GA. You decide to cross the threshold, but it's usually an easy decision to make.

### Cut as Needed

The last bit of advice I can give you about taking a product to GA is to never be afraid to cut. We become enamored with our vision for what the product can and should be. We think of all the critical customer asks that we haven't gotten to, yet. We defined an MVP, and there are still features from our minimal set that aren't there, yet, or we feel aren't viable. But if there are people saying they'll pay for it, as it is, and if you feel it can stand up to scrutiny in the market, don't hold back just because you didn't do everything you wanted or thought you should.

In the end, the only sure fire recipe is to do the work to make the product releasable at any moment; get feedback early and often; and let the market decide when you have a GA product instead of some arbitrary plan. The shape of the product may not be what you expected it to be, but if you take reasonable steps, you can remain agile, deliver more, and get a product out the door sooner than anyone--even you--expected.
