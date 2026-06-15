---
title: "Nobody talks about this"
date: 2026-06-15
author: Ibrahim Stephenson
categories:
  - Power Platform
  - Azure
  - ALM
tags:
  - power-automate
  - solutions
  - ALM
  - dataverse
draft: false
cover: 
---

There are a hundred videos about ALM. How to run a pipeline, which branching strategy to pick, the best way to structure your deployments. I have watched most of them. But almost nobody talks about how to structure your solutions.

<!--more-->

Maybe that is because most people already have it figured out inside a bigger company, with a platform team and a process they inherited. But if you are like me, owning the Dynamics system end to end, the full thing, all the weight on your shoulders, then this is something you actually have to think about. Something you have to sit down and restructure. And it ties in tight with how you decide to run your ALM.

Over my time as the developer at ThriveTribe I have been through four different solution strategies. All of them were viable in their own way. All of them worked until they didn't. I figured I would write them down in case someone else ends up in the same shoes I was once in.

## Strategy one: a new solution every sprint

At the very beginning there were no pipelines. Just import and export, sprint by sprint.

This was early in our Dynamics adoption. We were three developers and we made a new solution every sprint, and sprints were bi-weekly. All three of us worked out of the same sprint solution. No patches, no merges, everything in one place. The nice thing was transparency. Everyone knew who was working on what and when they were done, so we hardly ever clashed. Once the sprint closed we deployed to UAT and then PROD as managed, and started fresh the next sprint.

It was fine. To a point.

The problem crept up slowly. Doing it this way, the managed solution layers started to stack. Every sprint added another layer on top of the last, and after enough of them it got to a point that was just unmanageable.

## Strategy two: four core solutions and an abandoned pipeline

Around then the structure changed. There were redundancies, and a new offshore team came in with a different way of working.

Their idea was four core solutions, each in charge of its own components. One for flows, one for plugins, one for tables, one for security roles. Roughly. They tried to build a pipeline in Azure DevOps to tie it together. It just did not work. I am not sure why, I never saw the actual error, I was not very involved in that side of things back then. We trusted them to get it done. That turned out to be our mistake, because what we were left with was an abandoned pipeline.

The workflow on top of those solutions was patch based. You took a patch of whichever core solution you were working on, did your work, then carried that patch up to UAT and PROD as unmanaged. The patches were supposed to get merged back in by the pipeline. But the pipeline was abandoned, so they never did.

Nine months of this. Hundreds of patches. Literally hundreds. None of them merged, all of them unmanaged. It was a disaster sitting there waiting to happen.

That was about when I started taking control. I was comfortable enough in Power Platform by then. I had spotted the risks, I could see where it was all heading, and I decided enough was enough.

## Strategy three: one core solution per app, cleaned up by hand

So I gave it a week and a weekend of my own time to fix it properly.

I got rid of all the patches. I cleaned up UAT and PROD. Then I did some research and landed on one core solution per enterprise app as the way forward. Still no pipeline, because I could not find the time to configure it and the whole thing was new to me. So I went back to manual import and export through the PAC CLI. Not really manual, but manual in some sense.

The workflow was simple. Each developer made a new solution in DEV and did their work in there. Once they were done they added their components to the core solution, and the core solution got deployed. After deployment, the throwaway solutions got deleted to keep DEV clean. I will admit I am a bit of a neat freak, so having the solutions tidy and in order mattered to me.

This was going well. But it still felt like it was missing something.

## Strategy four: two core solutions, a hotfix, and a real pipeline

What it was missing was the pipeline. So I built one, and that is where I have landed.

Two core solutions and a third hotfix solution. Each one owns its components. Developers do not work directly in these, they spin up a new solution for their dev work, and once they are done they add it to the right core solution and I deploy it through the pipeline. It has been smooth so far, and for now I think this is the best fit for my organisation.

If you want the detail on how that pipeline actually got built, the XML surgery, the connection reference headaches, the layered architecture, I wrote the whole thing up in [How I Built a Proper ALM Pipeline for Power Platform](/posts/alm/).

## A note on team size

Keep in mind this is working for a team of five. If you are bigger than that, this probably is not the answer.

From what I have read, larger organisations sometimes give each team its own environment, sometimes each developer. If you are at that stage you most likely already have a tight ALM process and are not reading a post like this one.

## Where I want to take it next

I do not think strategy four is the finish line. It is just the best version I have reached so far, and every previous one felt that way too right up until it broke.

So I will turn the question around. If you run solutions at your own company, I would genuinely like to hear how. What you would change about this, what you think I am missing, whether there is a cleaner pattern I have not tried yet. The whole reason I wrote this down was the lack of anyone talking about it. So let's talk about it.
