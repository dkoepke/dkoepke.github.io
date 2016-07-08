---
layout: post-no-feature
title: "Inhuman Error"
description: "Nobody has ever caused an outage"
category: articles
tags: [management, operations, human error, mistakes, responsibility, accountability, outages, downtime, software engineering]
---

Any IT or operations person with long enough experience can tell you a horror story or two about an outage they caused. It is almost a right of passage in the industry: that first time your own fallibility sucker punches you in the gut and leaves you doubled over, tears stinging your eyes. But when someone tells me one of these stories, I can't shake the feeling that something is fundamentally wrong with how we talk about these things. The storyteller might recount the precise moment they made a critical mistake, and the incident report might have said the outage was due to, &ldquo;human error,&rdquo; but I think they're wrong.

Sure, humans make mistakes. We work when we are tired or sick or distracted by life. We miss critical details that we should have seen. We act recklessly or foolishly. We think we have checked every last detail before we push the button that brings down the whole site. We trust our health checks when they don't cover every case. No one should doubt the best and smartest human's capacity to do something uncharacteristically stupid.

But it is precisely the inevitability of human error that disqualifies it from consideration as a causitive factor. Since humans are doomed to make mistakes, we must mitigate the risks of those mistakes. We must implement safeguards. When human error is allowed to cause major issues, the safeguards have proven insufficient, and they are the ultimate cause.

## Uncommon affordances

Almost thirty years ago, Donald Norman's <cite style="font-style:italic;">The Psychology of Everyday Things</cite> discussed the affordances that mundane objects make for humans to use them correctly and safely. A door with a push plate and no handles tells a person that there is exactly one correct way of operating the door. Other doors have grip bars, but some of them want you to push and others want you to pull. The typical light switch is up when it is on, and down when it is off, and we can feel it click into place when toggled. Consumer electronics companies still insist on introducing fancy new light switches that offer no indication of what state the switch is in, and often do not give us feedback on whether we have really toggled it.

Our failure to use these everyday objects correctly is rarely disastrous. We get briefly frustrated with doors with handles want us to push them because they look like they should be pulled. We wonder if the lights have burnt out when we wave at a motion-sensing &ldquo;switch&rdquo; or if our gesture wasn't picked up or if our gesture was wrong or if the whole damnable thing is broken. **Well, even if it's working to spec, it's broken.**

Despite our everyday micro-frustrations with bad design, and our everyday reliance on good affordances, we rarely demand good design for our critical business operations. Upgrading a mission critical component should come with affordances that prevent data loss, outages, over-utilization of resources, and other bad outcomes. If we do not implement enough good affordances, a human will err in the course of their work.

## Double standards

The fallout from these inevitable errors is worth studying. I've spent my entire career in devops. At one time, these were distinct roles that, by background, I happened to bridge. I started programming when I was a little kid, and as I grew up and wanted to learn more, I got into Linux and which got me into systems administration. As the industry grew up, we realized that it caused more problems than it solved to isolate these responsibilities into specialist teams and the devops culture was born.

Walking this path has taught me a lot, but one of those lessons is that there's a terrible double standard regarding human error. If my role is a software engineer, and I make a mistake, I've introduced a bug. We look at why the tests didn't catch it. We create a ticket. We figure out what I did wrong, we fix it, and we move on. But we don't talk about it as, &ldquo;human error.&rdquo; We talk about it as a software defect.

If instead my role is in operations, my mistakes are human errors (i.e., my fault) and much discussed in that light. I am held responsible for all manner of problems with the architecture, even if a lot of bad decisions went into making a house-of-cards architecture. The absurdity of this is magnified when we consider Quality Assurance roles: there, my failure to catch a bug--the software engineer's error--is labeled as human error on my part. And we reach peak absurdity when we follow through on our devops culture. I'm relatively blameless when I introduce a bug that breaks functionality and entirely at fault if I reduce the capacity for the same functionality by assigning a new server the wrong role in our configuration management system. The bug probably took days or weeks to introduce; the operations error took a momentary lapse in a brief process.

Software engineers screw up all the time. We have killed people with our human errors. But the discussion doesn't quite get as personalized as it does for other roles, and it shouldn't. We can't be mad about the software engineer's treatment: the unfairness is that other people are not given the same understanding. Software is hugely complex, and mistakes will happen. But operations tasks aren't any different. When someone takes operations responsibilities, they are burdened with high risk factors, low reward, and the specter of the outage broadcast declaring, &ldquo;human error,&rdquo; as the cause.

## Striking out human error

If you can believe that the lack of affordances is the problem, and that we mistreat people when they take on operations task, then you can make one simple, big improvement: never attribute a problem to, &ldquo;human error,&rdquo; ever again. The cause is not the mistake the person made. The cause is what led to the human's mistake and what allowed that mistake to yield such bad results.

The focus on the mistake misses the point. Managers want to remediate problems. If the problem is defined as being human error, the results are silly and counterproductive. When a person has made a bad mistake is when they most need our support and understanding, when they most need us to shift the focus on how we, as a team, have allowed this to happen. Any good employee already feels awful about whatever they did wrong. They are beating themselves up. When we come along and identify their error as the cause, the remediation takes the form of managerial actions: stern talks, reduced access or responsibility, unnecessary mandatory re-trainings, even termination.

Never sacrifice a person to clarify the stakes to the team. A human sacrifice will always be more memorable than whatever point you were trying to make about SLAs.

## Co-pilots and auto-pilots

When we stop focusing on human error, we can start to apply the concept of design affordances to our business processes. To me, there are two broad categories of affordances we can give people that have operations responsibilities: co-pilots and auto-pilots.

Co-pilot affordances involve adding one or more additional humans to the process. The co-pilots provide checks and balances where a human is required to take risky, manual action. By adding other people to the process, we provide oversight that can correct human error before it causes problems. As risk increases, we need more co-pilots so that the chance of any one human making a disastrous error is reduced to the realm of unlikelihood.

The analogy to actual co-pilots within a cockpit is fairly obvious and leads to direct comparisons with pair programming, but I consider the category to include more than just the person that sits next to you during the risky action. Another type of "co-pilot" ensures the person charged with a task actually should be the one performing it. In some cases, this may be as simple as requiring operations people to take an internal training (or refresher course). In other cases, the co-pilot process may simply be a team check-in before a manual process to make sure that the people tasked with the process aren't mentally or physically exhausted, unhappy, or sick, and that they remember what the steps, risks, and emergency remediations are. While these slightly strain the "co-pilot" analogy, the focus remains on adding human verification, assistance, and support.

In contrast, auto-pilot solutions take control out of human hands entirely. A key insight of the infrastructure as code movement is that not only is a lot of the grunt work that operations people do suitable for automation, it is hugely beneficial to automate it. Any time there is a person following an operations manual (whether that manual has been written down or not), there is an opportunity to introduce an auto-pilot. Auto-pilot solutions can be as simple as a shell script that's manually invoked, but a Jenkins job, Chef cookbook, CloudFormation template, or the like is often a better choice.

The main problems with auto-pilots is that some tasks resist automation. We need more and better automation engineers, and the cooperation of people that build software and hardware, to achieve our version of the driverless car. Until then, many of us can only have a few auto-piloted processes that are kicked off after human intelligence validates that they are ready.

These sort of semi-automatic processes need to be viewed as an essential but transitionary step. The inputs to the auto-pilots that require human validation carry with them a risk of human error. Therefore, it is often the case that we use both co-pilots and auto-pilots as part of our operational affordances.

## Responsibility and accountability

None of this discussion is to suggest that people with operations responsibilities should not take responsibility for their mistakes and be held accountable for them. We want people that own up to their mistakes. We want the team (much more so than the manager) to hold each other accountable while remaining understanding and compassionate. Responsibility and accountability are necessary to have a healthy culture.

But attributing problems to human error does not create this culture. It creates a culture of blame; a culture where people can't focus on their tasks for worrying about the dire consequences of mistakes; a culture where people subconsciously opt out of sharing responsibility and accountability with their colleagues for fear of what it could mean for them.

The wrong mindset inhibits the creation of operational affordances, and the lack of operational affordances poisons the culture. Do not focus on human error. Nobody has ever caused an outage.
