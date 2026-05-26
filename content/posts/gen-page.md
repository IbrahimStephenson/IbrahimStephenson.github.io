---
title: "Generative Pages in Model-Driven Apps: A Practical Guide"
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

If you haven't been living under a rock you should have heard about generative pages by now. Essentially it's a full React page created using prompts, no code needed, no drag and drop. Nothing. Just talk to the AI and it will create a functional site.

<!--more-->

Now I, like many others, was quite intrigued and decided to give it a go. Whilst I was impressed by how it worked out of the box, it definitely wasn't production ready. Until I hooked it up to Claude Code inside VS Code, and boy oh boy was this thing amazing.

So I figured I would create a quick post explaining how to do that so everyone can fully utilise the true hidden potential of generative pages in model-driven apps.

## What Are Generative Pages?

Generative pages let you describe a page in natural language and get back a fully working React and TypeScript application that lives inside your model-driven app. No Power Fx. No drag and drop. Real code.

Model-driven apps are rigid by design. Custom pages improved that by embedding a canvas app inside the MDA, but you were still limited to what Power Fx could do. Generative pages remove that ceiling. They've been generally available since October 2025, support up to six Dataverse tables per page, sit inside your solution so they travel through your ALM pipeline, and require no extra credits or licences beyond what your users already have.

## Method 1: Building Through the Maker Studio

The native route is the quickest way to get started.

Open your model-driven app in Power Apps Studio and add a new generative page. You will get a split interface: a chat panel on the left and a preview on the right. Describe the page you want in plain English. The agent will produce a plan, flag any assumptions it has made, then generate the code and render a preview.

You can iterate directly from the chat panel. Ask it to adjust the layout, add a filter, change a component. The code is accessible inline if you want to inspect or edit it directly.

Once you are happy, save and publish. Add the page to your app navigation and it is live.

This is the fastest route for simple pages. For anything more complex, skip the studio entirely and build locally.

## Method 2: Building via Claude Code and the Power Platform Skills Plugin

This is the approach I used for the Support Hub rebuild. What it produces is a different class of result from the native experience.

You need three things. Claude installed on your machine. VS Code with the Power Platform Tools extension. PAC CLI authenticated to your environment.

### Installing the plugin

Open Claude in your terminal and type `/plugins`. Navigate to Add Marketplace and add the following:

```
/plugin marketplace add microsoft/power-platform-skills
```

Once the marketplace is added, find Power Platform Skills, open it, and install the Model Apps skill. Press Escape to exit the plugin menu.

### Generating the page

Run the skill with:

```
/gen-page
```

Claude will check prerequisites (PAC CLI, Node.js) and authenticate against your environment. It will then ask whether you want to create a new page or edit an existing one. Select create.

When prompted to describe the page, be specific. The more detail you provide about the data model, the filtering logic, the fields and relationships involved, the better the output. Include table names, field schema names, and exactly how you want the data to behave.

Claude will retrieve your Dataverse schema, generate the React and TypeScript code, and deploy it to your environment. This takes roughly 10 to 15 minutes and burns through a large chunk of your context window. Once deployed, it will offer to verify the page using Playwright, opening a browser session and testing the app feature by feature. If anything is not working, it will fix it before handing back to you.

For the Support Hub I described the full data model: the `ttd_supporthub` table, the service and programme lookups, the nullable programme field, the `hasascribe` flag, the search behaviour across title, description and keywords. What came back was a working five-screen React app with Fluent UI components, real-time search, and proper navigation via React state.

### Iterating after deployment

Once the page exists in your environment you can keep refining it through Claude. Describe the change you want, Claude modifies the component and redeploys via PAC CLI. No need to go back into Power Apps Studio.

When I wanted a Training Videos section added I described the screen, the video card layout, and the inline iframe player. Claude built it, deployed it, done. In the canvas app world that would have been a significant piece of work.

You can also open the generative page inside the MDA designer after deployment for inline edits if you prefer that interface.

## What to Know Before You Start

The prompt is everything. A vague description produces a vague result. You need to know your data model, your field names, and the exact behaviour you want before you start. The better you can describe the system, the better the output.

The code is real React and TypeScript. If something goes wrong or the output needs adjusting, you need to be able to read it or work with someone who can.

Every generated page is solution-aware. It deploys into your solution, moves through DEV, UAT, and PROD with your pipeline, and behaves like any other solution component.

Power Platform always said you didn't need to write code to build what you needed. Generative pages are the first time that has actually been true for me.
