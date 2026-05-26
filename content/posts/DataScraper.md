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

At my company, the Datascraper was a thing of legend. Believed to have been lost. Forgotten.

Until I got my hands on it.

<!--more-->

## The Original

Staff were receiving PDFs from GPs, healthcare professionals, schools. Patient referral documents. Every one had to be opened, read, and entered into Dynamics by hand.

Field by field. Record by record.

Then one of my previous colleagues built the Datascraper 1.0. It pulled information from the PDFs and pushed it into the database as referrals. No manual input. At the time it was the most impressive thing anyone on the team had seen.

Then the cracks appeared.

The tool had been trained on specific templates. Our staff, as staff do, started feeding it everything else. Documents with different layouts. Typos, missing fields, information in the wrong position. Working with patient data, inaccurate records aren't just an inconvenience. They follow people through their care. A missed field, a wrong value, a note in the wrong place.

The developer who built it left the company. The rest of us had a backlog. No one had time to fix it. So it got shelved. Unused. Forgotten.

I was in a meeting when someone mentioned it in passing. I remembered a document the developer had left behind, titled "AI DataScraper." After the meeting I went to find it.

That was my opening.

## The People's Hero

I spoke with my manager and put my idea forward: rebuild the Datascraper, but better. More accurate. Built on a model that could handle the variation we were actually dealing with. I was quickly shut down. More pressing items, bug fixes, BAU tasks, dev requests from ADOs. No time.

So I offered to do it in my own time.

I was determined to build something that wouldn't break when the next unusual document came through. Something that would grow alongside the company, not fall short of it.

The first thing I did was talk to the staff. Found out exactly what was coming in from GPs, clinicians, schools. Got a list from various sources, all formatted differently. One was a TIFF file. (Which is odd. But fine.)

I needed a solution that didn't depend on seeing the exact template before. One that could read a document the way a person would. Understand context. Find information because it knows what it's looking for, not just where it usually sits.

I landed on Claude via Anthropic's API, called through a HTTP action in Power Automate.

It reads a document and infers the fields itself. Different GP letterheads, different layouts, fields in different positions. It doesn't matter. Previously every field had to be manually mapped. Now a detailed prompt does that work.

GP notes are where this gets interesting. If you've worked with GPs you know they put everything in the notes. Free text. Unstructured. One clinician wrote that a patient smoked 15 times a day and listed several conditions alongside it. Claude picked that up, identified the relevant Dataverse fields, and populated them with the right data.

But confidence matters. Claude doesn't always get it right. So I built a scoring system.

For every field it interprets, Claude returns a score out of 10 with its reasoning. Anything below a 9, it does not populate the field. Instead it drops the information into the notes, then flags it in the final email to the advisor: the field, the score, why it wasn't confident. The advisor reviews it. The record stays clean.

Wrong patient data doesn't get a second chance to be corrected quietly. It either goes in right or it gets flagged.

## Generative Pages

The flow needed a way for advisors to actually submit these documents. We run Dataverse from a Model Driven App and there's no native PDF uploader.

That's where Generative Pages came in. Microsoft's release lets you generate a full React page inside an MDA. It bypasses the limitations of both Canvas Apps and Model Driven Apps entirely. I embedded a PDF uploader directly into the app for staff to use.

Then I hit a second problem. A lot of these documents weren't PDFs. Converting them manually would have created work, not saved it.

I added a second page: a PDF converter. Staff could drop in up to 20 .docx files at once, click convert, and the documents would come back as PDFs ready to upload. No manual conversion.

One more thing slowed me down. Every change to a Generative Page after the initial prompt rewrote the entire component. Small adjustments were taking too long. So I connected Claude Code directly to the Power Apps environment using the Power Platform Skills plugin. Described the changes I needed. Loading states, error handling, form resets. Claude modified the React component directly.

If you want to build your own Generative Page and connect it to Claude Code, I wrote a full guide: [How to Build a Generative Page with Claude Code](https://ibrahimstephenson.github.io/posts/gen-page/).

---

The original Datascraper was a good solution for the world it was built for. Fixed templates, predictable documents, a team that stayed inside the lines.

That world doesn't exist anymore.

Build for the variation, not the ideal case. The ideal case never shows up.
