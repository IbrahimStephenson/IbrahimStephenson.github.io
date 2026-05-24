---
title: "How I Built a Proper ALM Pipeline for Power Platform (Starting From Chaos)"
date: 2026-05-16
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - alm
  - azure-devops
  - pac-cli
  - dataverse
  - power-automate
draft: false
cover: /images/ALM.png
---

I used to dread our sprint deployments. Every 2 weeks we would deploy our solutions, individually, unmanaged. Chaotic. One person's change overwrote someone else's. Hotfixes, redeployments, environment restorations. It was chaos. Unmanaged chaos.

This is how I fixed it, and what it cost me.

<!--more-->

## Quick Backstory

It wasn't always this bad. Believe it or not, at one point we had a working managed ALM pipeline. Running on Azure DevOps through each environment, source control, the works. Then there were redundancies, inadequate documentation. Pressing business needs meant attention was forced elsewhere. Resources stretched thin. A new offshore team who had their own plan in mind.

Over time the pipeline kept failing. We didn't have the resources to fix it in time, we just had to keep moving. Deploying unmanaged, one solution at a time. At first we thought it was recoverable. It's fine, I'll make an unmanaged change this time but next time I'll do it properly. Spoiler: there was no next time. This was how it was, for a long while.

Then I got sick of the fear. I took ownership, something I should have done a long time ago. But just because I had a passion to fix it doesn't mean there was a business need, doesn't mean there was time to. I still had pressing developments that had to be shipped, deadlines I had to meet. So what did I do?

I worked weekends. Off hours. I took it upon myself to fix this, myself.

## The Plan

I needed a plan first. So I spoke with the best person in the industry. Parvez. If you don't know who he is, he is the ALM guru for Power Platform. I was fortunate enough that through my connections I could have a quick call with him. I explained my issue and he gave me a solution.

Step 1: locate one source of truth. One environment the rest has to reflect.
Step 2: get rid of deprecated flows, components, apps. Anything that does not have a purpose has to go. Clean it up.
Step 3: pipeline and source control.

And so that's exactly what I did. It was not easy. It was in fact the single most painful thing I've done. Scanning through hundreds of flows, components, apps. Pieces of work half finished. Trying to understand the purpose of it all. Collating a massive document with all my findings, each flow's purpose explained. If it had no purpose it had no place here and had to go. It was far from easy, but it was more than necessary. We needed a fresh start. I needed a fresh start.

## The Layered Architecture

Once I was able to get rid of all the junk and bloat in our PROD environment, came the time to clean up the rest. One fell sweep.

I created 4 core solutions. One for Components, one for Data, one for Flows, and one for UI. These 4 solutions contained all our customisations in PROD. I took this and passed it back into UAT.

It took a couple of hours and a few errors. Blocked by a third-party PCF that did not exist anywhere but was referenced in our main lead table. I had to request a single solution zip from the company that contained this component, after needing to explain the situation and a long unnecessary back and forth. Once I had this, there was another error. A field called startdate was defined as UserLocal datetime, but the UAT environment had it as DateOnly. Then a formula column called joinabledate had been built against the DateOnly version and was blocking the import. Since Dataverse doesn't let you change the DateTime behaviour on a column that has dependencies, the only way through was XML surgery. Opening the solution zip and locating every instance of startdate across multiple tables, correcting the behaviour values to match what the environment expected.

Once I was finally able to import into UAT, I did the same thing for DEV. After a weekend of testing and cross-referencing I was certain the 3 environments could be triplets. They were identical.

## The Pipeline

I built this pipeline using Azure DevOps. My reason was simple. There was more documentation online for it, more resources for me to reference and learn from. So it was a bit of a no brainer.

It ran from 2 YAML pipelines. The export pipeline runs against DEV: it installs PAC CLI, exports the solution XML, and commits it to Azure Repos. The deployment pipeline takes the XML, packs it as managed, and imports it to UAT with force overwrite.

No more manual imports. Solutions go to UAT and PROD via the pipeline only. No one is making changes to production anymore.

## What It Actually Took

Now I can sit back and enjoy our retros, our end of sprint ceremonies, our reviews. I can feel confident our deployments will work, the system will behave as expected. And better than everything, I can sleep peacefully every other Thursday.

---
