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

I used to dread our sprint deployments.

Every two weeks we deployed our solutions. Individually. Unmanaged. One person's change overwrote someone else's. Hotfixes, redeployments, environment restorations. It was chaos. Unmanaged chaos.

This is how I fixed it, and what it cost me.

<!--more-->

## Quick Backstory

It wasn't always this bad.

At one point we had a working managed ALM pipeline. Azure DevOps, source control, promotion through each environment. It was proper. Then people left. Documentation was inadequate. Business needs pulled attention elsewhere, resources stretched thin, and a new offshore team arrived with their own plan.

The pipeline kept failing. We didn't have time to fix it. So we kept moving. Deploying unmanaged, one solution at a time.

At first it felt recoverable. I'll do it properly next time. Spoiler: there was no next time. This was just how it was. For a long while.

Then I got sick of the fear. I took ownership, something I should have done a long time ago.

But wanting to fix it doesn't create the time to fix it. I still had developments to ship, deadlines to hit. So I worked weekends. Off hours. I took it upon myself, by myself.

## The Plan

I needed to talk to someone who actually knew what they were doing.

Parvez. If you don't know who he is, he's the ALM guru for Power Platform. Through my connections I managed to get a call with him. I explained the situation. He gave me three steps.

One: find a single source of truth. One environment everything else has to reflect.

Two: cut everything that has no purpose. Deprecated flows, dead components, unused apps. Gone.

Three: pipeline and source control.

Simple to say. It was in fact the single most painful thing I've done.

## The Cleanup

I started in PROD. Scanned through hundreds of flows, components, apps. Work half finished. Things built by people who had left years ago. No documentation, no context, just a name and a trigger and a guess at what it was supposed to do.

I built a document. Every flow, every component, its purpose explained. If it had no purpose it had no place. It had to go.

Weeks of this. Then I had four clean solutions. Components, Data, Flows, UI. Everything we actually used, nothing we didn't.

I took those four solutions and pushed them back into UAT.

Then the errors started.

A third-party PCF control referenced in our main lead table that didn't exist anywhere in our environments. I had to track down the company that built it, explain the situation, go back and forth longer than it should have taken, and get a single solution zip with the component inside.

Got that in. Another error. A field called `startdate` was defined as UserLocal datetime in PROD. UAT had it as DateOnly. A formula column called `joinabledate` had been built against the DateOnly version and was blocking the import. Dataverse won't let you change datetime behaviour on a column that has dependencies. The only way through was XML surgery. Opening the solution zip, locating every instance of `startdate` across multiple tables, correcting the behaviour values to match what the environment expected. One by one.

Then it imported.

I did the same for DEV. After a weekend of testing and cross-referencing, the three environments were identical. Triplets.

## The Pipeline

I built it on Azure DevOps. More documentation online, more to learn from. Easy choice.

Two YAML pipelines. The export pipeline runs against DEV: installs PAC CLI, exports the solution as XML, commits it to Azure Repos. The deployment pipeline takes that XML, packs it as managed, and imports it to UAT with force overwrite.

No more manual imports. No more unmanaged changes slipping through. Solutions move through DEV, UAT, and PROD via the pipeline only.

No one touches production directly anymore.

## What It Actually Took

The deployments work now. The system behaves as expected. I sit through our end of sprint reviews without that quiet dread of what might break at midnight.

And every other Thursday night, I sleep.

That's what it cost: weekends, off hours, XML I never wanted to read. That's what it bought: a system that doesn't require fear to operate.

Build the thing that makes the fear unnecessary. Even if no one asks you to.
