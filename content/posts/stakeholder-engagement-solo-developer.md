---
title: "Why Stakeholder Engagement is Half the Job When You're the Only Developer"
date: 2026-05-20
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - dynamics-365
  - stakeholder-engagement
  - solo-developer
cover: /images/stakeholder.jpg
draft: false
---

I am the only internal Power Platform developer at ThriveTribe. No team to bounce ideas off. No senior dev to escalate to. Just me, the platform, and 400 staff across 10+ NHS programmes who need things to work.

That setup teaches you something quickly. Technical skill gets you maybe halfway there. The other half is knowing what to actually build.

And you only find that out by talking to people.

<!--more-->

## Everyone Has a Different Lens

ThriveTribe has a CTO. It has programme managers, operations leads, clinical advisors, and frontline staff who open Dynamics 365 first thing every morning and close it last thing every evening.

Every conversation I have with each of those people is valuable. But they are valuable in completely different ways.

A conversation with the CTO gives me direction. What are the strategic priorities? Where is the organisation heading? What integrations are coming? That context shapes the decisions I make about architecture, solution design, and what to build for longevity versus what to build for now.

A conversation with a programme manager gives me scope. What is the process supposed to look like? What are the compliance requirements? What does success actually mean for this programme?

A conversation with a frontline advisor gives me reality. What is the process actually like? Where does it break down? What workarounds have they quietly built into their daily routine because the system doesn't quite fit the work?

That last one is the one developers most often skip. It is also the one that tells you the most.

## The Things You Miss From a Distance

A while back I sat with one of our advisors and watched them complete a task end to end. No instructions. No leading questions. Just watching.

Frustrated clicks that went nowhere. A field they always skipped. Uncertainty on the same button every time.

I went in thinking I was going to hear about a feature request. Instead I heard something more useful: the system was fighting them. Not in a dramatic way. In the quiet, daily, death-by-a-thousand-cuts way that never makes it into a support ticket but costs real time and real energy every single day.

Hearing their words changed mine. It was not "optimise the form." It was "make this screen stop fighting me."

So I set real defaults so they were not typing the same values every record. Reordered fields to match the way the work actually happens. Trimmed the command bar to the actions they use daily. No new features. A few hours of work. The app felt faster.

That insight did not come from a requirements document. It came from sitting with someone and shutting up long enough to see what they were actually dealing with.

## Why Every Level Matters Equally

It is tempting as a developer to weight the conversations differently. The CTO sets the agenda, so that conversation feels more important. The advisor is just a user.

That framing is wrong, and it will cause you to build the wrong things.

The advisor is the person your work has to serve. They are the ones who will tell you that the new referral form is technically complete but has a dropdown that takes four clicks to get to when the work demands it be the first thing on screen. They are the ones who will tell you that a field you spent a week building is always left blank because nobody actually needs it.

The CTO will rarely know that. It is not their job to know that.

My job, as the only developer, is to hold both of those conversations and build something that satisfies both. Strategic direction from the top. Ground truth from the people doing the work. Those two things do not always point in the same direction, and when they diverge it is worth raising before you write a single line of code.

## What Good Stakeholder Engagement Actually Looks Like

It is not a kickoff meeting where you collect requirements and disappear. It is ongoing.

It is checking back in with an advisor two weeks after a change to ask if it actually helped. It is being present enough in the organisation that people flag things to you informally, not just through a formal ticket. It is translating what a CTO describes in strategic terms into something a canvas app can actually deliver, and then validating that translation with the people who will use it daily.

As the only developer, I do not have the luxury of specialising in just the technical side. The business analysis, the user research, the stakeholder management — that is all part of the role too.

The best thing I have built at ThriveTribe was not the most technically complex. It was the one where I understood the problem well enough before I started that I barely had to change anything after deployment.

That only happened because I asked the right people the right questions first.
