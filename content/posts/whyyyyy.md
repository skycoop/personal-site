---
title: "Whyyyyy!?"
date: 2022-12-26T13:25:43-07:00
draft: true
toc: false
images:
tags:
  - Incident Analysis
  - Learning from Incidents
  - Reliability
---

The [2022 VOID Report](https://www.thevoid.community/report) is out just in time for me to read over it during the winter break (seriously great timing guys). One of the key findings I found particularly interesting during my read-through was a decline in Root Cause Analysis (RCA). I for one celebrate this change, and a no small part of excitement is rooted in my extreme dislike for the Five Why's method of performing RCA. So in the spirit of letting old things go as we move into a New Year, here's my critique of Five Whys analysis from my experience using it for 3 years as an SRE at Quizlet. And if all goes to plan, I will never have to deal with it again after this (more on that later).

## Why #1: It doesn't scale well

Five Whys analysis works great if every question has one answer, and each answer begets a single question. However, this doesn't map well onto reality, especially when we're attempting to analyze complex, distributed applications written and run by humans. Every failure we encounter will have a complex web of intersecting technical and organizational causes.

If you try to represent your analysis as a tree of questions and answers to cope with this (as Quizlet did), you'll quickly find that it gets bloated. I don't even want to know how many hours of my life were lost to wrangling with list formatting in Google Docs. More importantly, this results in an analysis that has a hard time representing failure modes that are related since they can end up in separate branches of the tree.

In addition it's a very inflexible process which limits it's applicability and ease of use.

## Why #2: It's shockingly subjective

On it's surface, the Five Whys feels like a method that should be extremely **objective**. It has echoes of the Socratic Method, that truth can be reached by asking questions until we reach the core of an issue. However, in every analysis I performed using it, I would always find a comment asking why I didn't ask a particular question or answer in some other way. And there's the rub. Everyone will come at the Five Whys analysis from a different viewpoint. The questions that I as an SRE would ask about an incident won't line up with that of an developer, and definitely won't line up with that of a manager or executive.

In my experience, this ends up causing a couple patterns of behaviour that end up really damaging the final result. One is an attempt by authors to try to capture every single possible question in advance, resulting in large, hard-to-follow trees of analysis. The second is far more insidious, an assumption that this "objective" analysis captured everything that's important. At Quizlet we discovered this was untrue when we added some additional sections to our incident analysis template: "What went well?", "Where did we get lucky?", and "What could have gone better?". These sections ended up capturing some of the more important learnings from our incidents and were far easier to write than the Five Whys.

## Why #3: Action items

An idea that naturally flows from a Five Whys analysis is that all terminating answers should have a corresponding action item that's designed to address them. However, this has actually backfired at Quizlet. We have hundreds of outstanding action items that are out-of-date and overwhelm the teams they are assigned to. Part of the issue is that the tickets end up being fairly granular solutions that feel too low value for teams to prioritize. In other cases, it comes down to the fact that that specific terminating condition isn't actually what we need to be focused on and the team instead chooses to focus on higher level reliability fixes that moot whatever the specific root cause was for this incident.

## Why #4: Chasing a mirage

There's a fantastic article by Lorin Hochstein called [Root cause of failure, root cause of success](https://surfingcomplexity.blog/2021/08/13/root-cause-of-failure-root-cause-of-success/) that articulates the key idea better than I ever could. But if I were to take a stab, the core misunderstanding of RCA as a whole is that our systems were in a stable, passive state before the incident and then _something_ happened to destabilize them. But in fact this stability often arises from a set of active processes that work together to keep things stable.

In the first view of the world, knowing and understanding that _something_ will allow us to craft responses to ensure that something will never again destabilize the system. This is what leads to RCA being such an attractive method for learning from incidents. However, in the second view, that _something_ while important, is not where analysis should end. Instead we need to take a look at what caused the collective failure of the supporting processes that allowed the trigger to cause the incident.

This is often a much more complex analysis to perform, but when done correctly can result in a system designed not only to counter the specific defect at the core of any singular failure but to counter unforeseen defects that could have caused a similar collapse of the stabilizing forces in the system.

## Why #5: Meta-analysis

That last point leads into the last deficiency I see in the Five Whys, namely it's limited to the cope of a single incident. It's not a framework that can scale to meta-analysis of multiple incidents. Often a company's incidents will all share a core set of themes (remember that word for later) that repeatedly surface in incident after incident. Five Whys are unlikely to catch these in a consistent way that becomes easy to stitch together into a pattern that indicates larger organizational change is required.

## Enough whys, what now?

The answer we've settled on at Quizlet is an approach based on the [Howie Guide](https://www.jeli.io/howie/welcome). Namely, by having a collaborator tag incident data and talk with responders, you can identify topics of interest that can then be refined down to "themes". The fantastic part of thinking about incidents this way is that it counters each of the above issues I've identified.

1. It's very flexible and works for various types and scales of incident analyses
2. It embraces the subjectivity of the process
3. Action items are derived from what would best address the theme instead of a specific failure mode
4. It is more likely to detect the larger systemic issues
5. It applies as well to meta-analysis as it does a singular incident

As we move into 2023, Quizlet will standardize on this approach and with any luck kill the Five Whys once and for all.
