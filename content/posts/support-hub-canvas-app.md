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

We adopted Power Platform fast. Once the execs heard about it, the money we could save by migrating over, that was all it took. Next thing you knew we had hundreds of staff members using a half finished site. Hundreds of tickets rolling in per day, backlog as high as Everest. There was no way we were ever going to stop drowning.

But most of the tickets were due to incorrect use of our platform. Advisors not knowing where to go and what to press. It wasn't so clear to them since they were new to using Dynamics. They didn't understand it properly, nobody really did. Except for the devs.

So I figured I could fix one of our problems. Which would help fix the rest.

If I could make a centralised hub full of docs, support guides, videos, FAQs, host it on Dynamics, the platform our advisors were having problems with, then this would reduce our ticket submissions by a large margin. Which would relieve some pressure so we could focus on the important stuff.

And so that's what I did.

## The Problem

The requirements were to have a hub that could be updated by managers with docs and training guides as and when. But each guide would only apply to certain services or business units, while others applied universally.

The backlog wasn't just a number. Every ticket was a conversation we had to have manually, triaging the same questions, pointing people to the same places, every single day. We needed something that could absorb that load without creating new work to maintain it.

The obvious answer would have been a SharePoint library. But that would have needed developer involvement every time something changed, which in a fast-moving organisation with 10+ services is basically constant. I needed something non-technical staff could maintain themselves, that surfaced the right content for the right service and programme, and that didn't need a code change every time a new guide appeared.

## The Build

### The data model

The content is stored in a custom table called SupportHubs. Each record has a URL, keywords, description, service and programme lookup, and a title. The keyword and description fields are used for categorical distinctions and the search functionality.

The programme lookup is nullable by design. No programme set means the guide surfaces for every programme under that service. Programme set means it only appears for that one. That single decision is what makes the whole thing flexible enough to grow without constant maintenance.

### The app

Four screens, embedded in Dynamics.

Home screen had three options: Scribe Guides, Raise a Ticket, Training Videos.

The Scribe Guides option took advisors to a service selection gateway. Upon selection it shows a list of programmes associated to the service, to help further distinguish what guide the user is looking for. The guide list then filters the SupportHub table where the service matches and either the programme field is blank or matches what was selected. A search function runs across description and keywords in real time as the advisor types. Each result has an Open Guide button that launches the URL.

## Self-Scalability in Practice

After deployment I ran a training session showing lead staff how to add a guide themselves.

Adding a new guide is: open the Model-Driven App, create a Support Hub record, fill in the title, description, keywords, URL, pick the service and programme. It is live in the Canvas App immediately. No developer. No deployment. No code change.

New service onboarded? Flip the flag. New programme added? It appears automatically because the gallery pulls from the existing Programmes table. Guide updated? Paste the new URL into the record.

## The Result

Ticket volume dropped, quick. Team leads took accountability for their guides staying up to date. We ran regular sessions with them on new features and they started creating service-specific guides to share with their own teams. The system absorbed work that had previously landed on us.

That was always the goal. Build something that makes you redundant for maintenance, not something that creates a permanent dependency on you.
