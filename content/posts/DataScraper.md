---
title: "The Datascraper"
date: 2026-05-09
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - power-automate
  - ai-builder
  - claude
  - dataverse
draft: false
featured: true
cover: /images/datascraper.png
---

We had a solution everyone loved. Until they didn't. Here's what happened, and what I built to replace it.

<!--more-->

## The Original

Staff were manually typing data from referral forms into Dataverse. Same fields every time. Patient name, date of birth, NHS number, GP surgery. Over and over, from documents that already had all of that written on them.

So I built something. We called it the Datascraper.

It used AI Builder's document processing model. You train it on example documents, tell it which fields to extract, it learns the layout. Staff upload a PDF, the model reads it, the data lands in Dataverse automatically. No typing.

People loved it. That does not happen often with back-office tools. It felt like a proper win.

Then it started breaking.

## What Actually Went Wrong

The issue was built into how document processing models work. You train them on specific layouts. The model learns where things usually appear on the page. That works great until someone updates their referral template, which GP surgeries do. New services came with different paperwork. Forms got redesigned.

Every time the layout shifted, even slightly, the model started returning wrong data. Fields came back blank. Sometimes it confidently filled in the wrong thing with no error. Staff stopped trusting it. Then they stopped using it. Then it became the thing people quietly worked around.

I knew it needed replacing. I just did not know with what yet.

## Generative Pages

I had been reading about generative pages in Power Apps. The idea is you describe what you want, and it generates a working React app inside the designer. I was sceptical. I have seen enough AI-generated interfaces to expect something generic and broken.

I tried it anyway.

Honestly? Better than I expected. Way better. I had a working front end in one session. File upload, a service selector pulling live from Dataverse, submit button, loading states, success and error feedback. Clean code I could iterate on just by describing what I wanted changed. No dragging components around a canvas.

That was the front end sorted. I still needed something to actually read the documents.

## Where Claude Comes In

This is the part that changed everything compared to the original Datascraper.

Instead of training a model on fixed layouts, I built a Power Automate flow called Process Referral Document. When a file gets uploaded, the flow sends the PDF to Claude via an HTTP action along with a prompt that says, roughly: this is a referral for this specific service, extract these fields, return it as JSON.

Claude reads it the way a person would. It understands context. It does not need to have seen that exact template before. Different GP letterheads, different formats, fields in different positions. It does not matter. It finds the information because it knows what it is looking for, not just where it usually appears.

The flow then maps that JSON to the right Dataverse fields and creates the records. Staff get an email with direct links to review what was created.

The whole thing is genuinely more reliable than the original, with less setup and zero retraining when a new document format comes in.

## Then Claude Helped Build the Front End Too

This bit I still find a bit mad.

When I needed to refine the generative page, I connected Claude Code directly to the Power Apps environment using the Power Platform Skills plugin. I described the changes I wanted. Claude modified the React component directly. Loading states, error handling, form resets. Done.

So Claude is doing two things in this solution. Reading the PDFs inside the flow. And he built the interface staff use to upload them in the first place.

## The Difference

The old Datascraper needed training data, layout-specific configuration, and regular maintenance every time a template changed. When it broke, it broke silently. Wrong data, no error, no indication anything had gone wrong.

The new version needs none of that upfront. The prompt defines what to extract. New service in scope? Update the prompt and the field mapping. That is it.

The Datascraper was a good solution. It just had a ceiling. This one does not.

---

*Built on Power Automate, Dataverse, Power Apps generative pages, and Claude via Anthropic API.*