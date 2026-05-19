---
title: "Building a Self-Service Knowledge Portal That Runs Itself"
date: 2026-05-20
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - canvas-apps
  - dataverse
  - power-apps
  - dynamics-365
draft: false
---

The worst kind of support ticket is the one that didn't need to be raised.

<!--more-->

## The Problem

Advisors across ThriveTribe's NHS services were raising tickets for things they could answer themselves with the right guidance in front of them. How to complete a referral in Dynamics. How to log a clinical milestone. Basic system navigation questions. The kind of thing that should be self-serviceable.

The issue wasn't just ticket volume. It was that any existing documentation was scattered, hard to find, and not specific to the service or programme the advisor was working on. A smoking cessation advisor and a weight management advisor have different workflows, different forms, different steps. Generic documentation wasn't cutting it.

The obvious solution would have been a SharePoint document library. But that would have required developer involvement every time something changed, which in a fast-moving organisation with 10+ services is constant. I needed something that non-technical staff could maintain themselves, that surfaced the right content for the right service and programme, and that didn't require a code change every time a guide was added or updated.

## Thinking Through the Architecture

The requirement that shaped everything was self-scalability. Whatever I built had to be maintainable by a team lead with no technical background. That immediately pointed toward Dataverse as the content store rather than anything file-based. Records are easy. Deployments are not.

I also knew the tool had to live inside Dynamics 365 rather than being a separate portal. Advisors are already in Dynamics all day. Adding another URL to go to means another login, another context switch, another reason not to use it. A Canvas App embedded in the Model-Driven App meant staff were already authenticated in the right context with no friction.

The question was how to handle the fact that some guides are relevant to all programmes on a service while others are specific to one programme. I didn't want to duplicate records, and I didn't want to hardcode anything.

The answer was a nullable programme lookup.

## The Build

Layer 1: Dataverse Schema

The content store is a custom table, ttd_supporthub. Each record holds a title, a plain-text description, comma-separated keywords for search, a URL to the actual guide, a service lookup, and a programme lookup.

The programme lookup is nullable by design. A guide with no programme set surfaces for all programmes under that service. A guide with a programme set only appears for that specific programme. One table, two behaviours, zero hardcoding. That single design decision is what makes the whole system flexible enough to scale without maintenance overhead.

There is also an isfeatured boolean for surfacing priority content, and the service table has a hasascribe boolean that controls which services appear in the app at all. Onboarding a new service means setting that flag. No app changes required.

Layer 2: The Canvas App

Four screens. Embedded in Dynamics so advisors never leave the environment they are already in.

The home screen gives two options: Scribe Guides or Raise a Ticket. The ticket button links directly to the Freshdesk portal, so the app acts as both a self-service tool and a gateway to formal support when self-service genuinely isn't enough.

The service selection screen shows only services where the hasascribe flag is true, pulled from a filtered Dataverse view. Selecting a service stores the choice in a variable and moves forward.

The programme selection screen filters to programmes linked to the selected service. Selecting one stores it and moves to the guide list.

The guide list is the key screen. It filters the supporthub table where the service matches and either the programme field is blank or matches the selected programme. A Search() function runs across description and keywords in real time as the advisor types. Each result has an Open Guide button that launches the linked URL.

## Self-Scalability in Practice

After deployment, adding a new guide is: open the Model-Driven App, create a Support Hub record, fill in the title, description, keywords, and URL, pick the service and programme from lookup dropdowns. It is live in the Canvas App immediately. No developer. No deployment. No code change.

New service onboarded? Set the Has a Scribe flag. New programme added? It appears automatically because the programme gallery pulls from the existing Programmes table. Guide updated or moved? Paste the new URL into the existing record.

The system scales with the organisation without ever needing to involve me again. That was the goal. A developer should build things that make themselves redundant for maintenance, not create ongoing dependency.

## The Result

Support ticket volume for self-serviceable queries dropped. Team leads now manage their own content libraries. The tool has been adopted across multiple services without any changes to the app itself since deployment.

And every time a new programme gets added to the organisation, it appears in the portal automatically.
