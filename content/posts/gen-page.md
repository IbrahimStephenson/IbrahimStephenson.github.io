---
title: "I Rebuilt the Support Hub With Generative Pages and It's Not Even Close"
date: 2026-05-02
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - canvas-apps
  - generative-pages
  - power-apps
  - claude
  - dataverse
draft: false
cover: /images/genpage.png
---

The Support Hub canvas app worked. It did the job. But I always knew it had a ceiling.

Then I discovered generative pages. And I rebuilt the whole thing in a fraction of the time, with a better result.

<!--more-->

## What We Had

The original Support Hub was a four-screen canvas app embedded in Dynamics 365. Advisors could browse Scribe guides filtered by service, search by keyword, and raise a Freshdesk ticket if they could not find what they needed. It was solid. People used it.

But canvas apps have friction. Formula language, screen navigation, gallery delegation limits, component quirks. Every change is a deployment. Every new requirement means opening Power Apps Studio and wrestling with something.

I had been watching generative pages develop quietly in the background. The idea is you describe what you want in plain English, and Power Apps generates a working React app directly inside the designer. Real code. Real components. Not a low-code approximation of an app. An actual app.

I was curious enough to try it for something real.

## The Rebuild

I wrote a detailed prompt describing the Support Hub: the screens, the data model, the filtering logic, the search behaviour, the Freshdesk link. The ttd_supporthub table, the service and programme lookups, the nullable programme field, the hasascribe flag on services.

What came back was a working React app. Five screens navigated via React state with no page reloads. Service selector, programme selector, guide list with real-time search across title, description and keywords, a home screen with action cards. Fluent UI components throughout so it looked native inside Dynamics.

The canvas app had taken multiple sessions to build and debug. Formula errors, delegation warnings, lookup comparison issues that took hours to track down. The generative page got there in one prompt, with cleaner code underneath.

But the part that actually surprised me was what came next.

## Bringing Claude Code In

Once the generative page existed, I connected Claude Code to the Power Apps environment using the Power Platform Skills plugin. Instead of manually editing the React code through the designer, I could just describe changes and Claude would modify the component directly.

That changed the iteration speed completely. I wanted a Training Videos section added, something the original canvas app never had. Advisors would be able to browse Microsoft Stream training videos and watch them inline in a modal without navigating away from the app. I described it. Claude built it. New screen, video card grid, searchable, inline iframe player.

In the canvas app world that would have been a significant piece of work. With Claude Code and the generative page, it was a conversation.

The deployment went through PAC CLI. Claude ran the commands. The app went to the environment. Done.

## What Actually Changed

The canvas app was four screens. The generative page is five, with training videos as a first-class feature.

The canvas app had delegation limitations on search. The React app does not have the same constraints.

The canvas app required Power Apps Studio for every change. The generative page can be iterated on through Claude Code from the terminal, described in plain English, deployed via PAC CLI.

The canvas app was something I built and maintained. The generative page is something I described and directed.

That distinction matters more than it sounds. As the only developer in the organisation, the less time I spend in Power Apps Studio wrestling with formula syntax, the more time I have for things that actually need thinking. Generative pages do not replace the thinking. They just get out of the way of it.

## Honest Caveat

It is not magic. The prompt matters. A vague description gets a vague result. I had to know the data model, the field names, the filtering logic, the exact behaviour I wanted. The better I could describe the system, the better the output.

And Claude Code editing the React component directly only works as well as your ability to tell it what you want. You still need to know what good looks like.

What changed is not that the knowledge requirement went away. It's that the execution gap between knowing what to build and having it built got much, much smaller.

---

*Built with Power Apps generative pages, Claude Code, Power Platform Skills plugin, and PAC CLI. Data from Dataverse.*