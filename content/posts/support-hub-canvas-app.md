---
title: "Building a Self-Service Knowledge Portal That Runs Itself"
date: 2026-04-18
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - canvas-apps
  - dataverse
  - power-apps
  - dynamics-365
draft: false
cover: /images/supporthub.png
---

The worst kind of support ticket is the one that didn't need to be raised.

<!--more-->

## The Problem

Advisors were raising tickets for things they could answer themselves. How to complete a referral in Dynamics. How to log a clinical milestone. Basic navigation questions. The kind of thing that should be self-serviceable.

The issue wasn't just volume. It was that any existing documentation was scattered, hard to find, and not specific to the service or programme the advisor was actually working on. A smoking cessation advisor and a weight management advisor have completely different workflows, different forms, different steps. Generic documentation wasn't cutting it.

The obvious answer would have been a SharePoint library. But that would have needed developer involvement every time something changed, which in a fast-moving organisation with 10+ services is basically constant. I needed something non-technical staff could maintain themselves, that surfaced the right content for the right service and programme, and that didn't need a code change every time a new guide appeared.

## Thinking Through the Architecture

The requirement that shaped everything was this: whatever I built had to be maintainable by a team lead with no technical background.

That immediately ruled out anything file-based. Records are easy. Deployments are not. Dataverse as the content store made sense straight away.

I also knew it had to live inside Dynamics 365 rather than being a separate portal. Advisors are already in Dynamics all day. Another URL means another login, another context switch, another reason not to bother. A Canvas App embedded in the Model-Driven App meant staff were already authenticated, already in the right place, zero friction.

The trickier question was how to handle guides that apply to all programmes on a service versus ones specific to a single programme. I didn't want to duplicate records and I didn't want to hardcode anything.

The answer was a nullable programme lookup. One field. Two behaviours. No hardcoding.

## The Build

**Dataverse Schema**

The content store is a custom table called ttd_supporthub. Each record has a title, a plain-text description, comma-separated keywords for search, a URL to the actual guide, a service lookup, and a programme lookup.

The programme lookup is nullable by design. No programme set means the guide surfaces for every programme under that service. Programme set means it only appears for that one. That single decision is what makes the whole thing flexible enough to grow without constant maintenance.

There's also an isfeatured boolean for priority content, and the service table has a hasascribe boolean that controls which services appear in the app at all. Onboarding a new service means flipping that flag. No app changes required.

**The Canvas App**

Four screens, embedded in Dynamics.

Home screen: two options. Scribe Guides or Raise a Ticket. The ticket button links straight to Freshdesk, so the app is both a self-service tool and a gateway to formal support when self-service genuinely isn't enough.

Service selection: shows only services where hasascribe is true, pulled from a filtered Dataverse view.

Programme selection: filters to programmes linked to the chosen service.

Guide list: filters the supporthub table where the service matches and either the programme field is blank or matches what was selected. A Search function runs across description and keywords in real time as the advisor types. Each result has an Open Guide button that launches the URL.

## Self-Scalability in Practice

After deployment, adding a new guide is: open the Model-Driven App, create a Support Hub record, fill in the title, description, keywords, URL, pick the service and programme. It is live in the Canvas App immediately. No developer. No deployment. No code change.

New service onboarded? Flip the flag. New programme added? It appears automatically because the gallery pulls from the existing Programmes table. Guide updated? Paste the new URL into the record.

The system grows with the organisation without ever needing to involve me. That was always the goal. Build something that makes you redundant for maintenance, not something that creates a permanent dependency on you.

## The Result

Ticket volume for self-serviceable queries dropped. Team leads manage their own content libraries. The tool has been adopted across multiple services without a single change to the app itself since deployment.

And every time a new programme gets added, it shows up in the portal automatically.