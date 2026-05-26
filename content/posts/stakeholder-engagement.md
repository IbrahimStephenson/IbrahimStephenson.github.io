---
title: "Why Stakeholder Engagement is Half the Job When You're the Only Developer"
date: 2026-04-11
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

I am the only internal Power Platform developer at ThriveTribe. No team to bounce ideas off. No senior dev to escalate to. Just me.

You learn something quickly in that setup. Technical skill gets you halfway there. The other half is knowing what to actually build.

And you only find that out by talking to people.

<!--more-->

## Everyone Has a Different Lens

ThriveTribe has a CTO, programme managers, operations leads, clinical advisors, and frontline staff who open Dynamics 365 first thing every morning and close it last thing at night.

Every conversation I have with each of them is valuable. But in completely different ways.

The CTO gives me direction. Strategic priorities, where the organisation is heading, what integrations are coming. That shapes how I think about architecture and what to build for longevity versus what to ship now.

Programme managers give me scope. What is the process supposed to look like? What are the compliance requirements? What does success mean for this programme specifically?

And then there are the frontline advisors. What I get from them is not requirements. It is reality. Where does the process actually break down? What workarounds have they quietly built into their day because the system doesn't quite fit how the work flows? That last one is the conversation developers most often skip. It is also the one that tells you the most.

## The Things You Miss From a Distance

A while back I sat with one of our advisors and watched them complete a task end to end. No instructions, no leading questions. Just watching.

Frustrated clicks that went nowhere. A field they always skipped. The same button causing uncertainty every single time.

I went in expecting a feature request. What I got was more useful than that. The system was fighting them. Not in a dramatic way. In the quiet, daily, death-by-a-thousand-cuts way that never makes it into a support ticket but costs real time and energy every single day.

So I set real defaults so they stopped typing the same values into every record. Reordered fields to match how the work actually happens. Trimmed the command bar down to the actions they actually use. No new features. A few hours of work. The app felt faster.

That insight did not come from a requirements document. It came from sitting with someone and shutting up long enough to see what they were dealing with.

It is tempting to weight these conversations by seniority. Resist that.

## Why Every Level Matters Equally

The CTO sets the agenda so that conversation can feel more important. The advisor is just a user.

That framing will cause you to build the wrong things.

The advisor is the person your work has to serve. They are the ones who will tell you the new referral form is technically complete but has a dropdown that takes four clicks to reach when the work demands it be the first thing on screen. They are the ones who will tell you that a field you spent a week building is always left blank because nobody actually needs it.

The CTO will rarely know that. It is not their job to know that.

My job is to hold both conversations and build something that satisfies both. Strategic direction from the top. Ground truth from the people doing the work. Those two things do not always point in the same direction, and when they diverge it is worth surfacing before you write a single line of code.

## What It Actually Looks Like in Practice

It is not a kickoff meeting where you collect requirements and disappear.

Two weeks after a change, I check back and ask if it actually helped. People flag things to me informally, not just through a ticket, because they know I want to hear it. When a CTO describes something in strategic terms, I have to translate that into something a canvas app can actually deliver, then validate that translation with the people who will use it every day.

As the only developer, I do not have the luxury of specialising in just the technical side. The business analysis, the user research, the stakeholder management — that is all part of the role too.

The best thing I have built at ThriveTribe was not the most technically complex. It was the one where I understood the problem well enough before I started that I barely had to change anything after deployment.

When there is no team to catch your mistakes, you cannot afford to build the wrong thing. So you talk to people first.
