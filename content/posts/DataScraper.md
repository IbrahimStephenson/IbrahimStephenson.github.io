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

At my company, "the Datascraper" was a thing of legend, believed to have been lost and forgotten. Until I got my hands on it.

<!--more-->

## The Original

Staff were manually inputting data from PDFs received from GPs that contained patient data into Dynamics. (TW) *Manually*.

Then one of my previous colleagues created the Datascraper 1.0. This essentially took information from PDFs and uploaded them as referrals into the database. At the time, revolutionary, groundbreaking. Then the cracks started to show.

Our staff were pushing it to its limits, and beyond. They would insert documents that differed from the trained templates. It would have typos, missed information, and fields inputted in the wrong place. As you can imagine, working with patient data this was a massive problem. However the developer in charge had left the company by then, and the rest of us had a backlog to work through. We didn't have time to fix this. So it was dumped, unused, forgotten.

Until I was in a meeting and it got brought up in passing. I remembered an old doc the developer left behind titled "AI DataScraper." After the meeting I went to look at the doc. Then an idea was born.

I spoke with my manager and put my idea forward: recreating the AI Datascraper, but better. More accurate. Trained using the best AI model to date. However I was quickly shut down, despite there being a need for this solution. There were more pressing items that needed attention. Bug fixes, BAU tasks, dev requests from ADOs. There was simply no time to work on it. So I offered to do it outside work hours. Took it upon myself as a personal project and challenge. I was determined to create something for everyone, something that won't break, that will grow alongside the company, not fall short from it. And that's exactly what I did.

## The People's Hero

The first thing I did was talk to the staff. Figure out what docs they were receiving from the GPs, healthcare professionals, schools, etc. I managed to get a list from various sources, all a bit different and unique. I even got a TIFF file.

Then I searched for a solution. Not just one that fits the requirements, but the best solution. A solution that will never break no matter what file is presented. A solution that is intuitive, can use semantic language, can understand more than just a prompt.

I landed on Anthropic's model. I used a HTTP action to speak to the AI using an Anthropic API. This was genius. Claude reads it the way a person would. It understands context. It does not need to have seen that exact template before. Different GP letterheads, different formats, fields in different positions. It does not matter. It finds the information because it knows what it is looking for, not just where it usually appears. Previously fields had to be manually mapped over. Now, with a detailed and high level prompt, I could get Claude to infer the fields itself.

GP notes, if you've ever worked with GPs before you know they love putting information in the notes. This information could map over onto possible fields on our Dataverse. But how do you map a free text note field onto a specific field? Well, with the right prompting we can get Claude to infer where it should land and insert it himself.

For example, one clinician wrote that a patient smoked 15 times a day and had certain conditions. Claude could pick this up, locate the field on Dataverse, and populate it with the accurate data.

But what if the data fields do not match? Well, that would be a problem. That's why I devised a scoring system. Out of 10, with reasoning, Claude explains how confident he feels with these interpretations. Anything less than a 9, he does not populate the field. Instead he inserts the info into the notes for the client, then flags to the advisor in the final email those fields, the score, and the reasoning behind it.

## Generative Pages

So I've briefly explained the flow, but how did the advisors input these PDFs in the first place? We ran Dataverse from an MDA and there is no native PDF uploader. That is where Generative Pages comes into play.

This release from Microsoft allows us to generate a full React page in our MDA. It's essentially a code app. It completely bypasses the natural limitations both Canvas Apps and Model Driven Apps have. It allows us to embed a beautifully designed, dynamic PDF uploader page into our MDA for our staff to use.

However, this is where I found my second issue. A lot of these documents were not in PDF format. I can't expect staff to spend time converting them into PDFs themselves. The whole point is to save time, not create more.

So I added a second page: a PDF converter. This allowed staff to input as many as 20 .docx files at once, click convert, and in a short span of time the documents would be converted into PDFs, downloaded locally onto the device. Which can then be uploaded onto the Datascraper page.

However the limitation with generative page is each change needed after the initial prompt rewrote the entire page. This took way too long, so I connected Claude Code directly to the Power Apps environment using the Power Platform Skills plugin. I described the changes I wanted. Claude modified the React component directly. Loading states, error handling, form resets. Done. 


The Datascraper was a good solution. It just had a ceiling. This one does not.

---