---
title: "Building a 14-Page NHS Self-Referral Form with 109 Fields and Real-Time Postcode Checking"
date: 2026-05-20
author: Ibrahim Stephenson
categories:
  - Power Platform
  - Azure
tags:
  - plumsail
  - power-automate
  - azure-functions
  - dataverse
  - javascript
draft: false
---

This one took a while to build. It also took a while to fully understand what I was building before I started.

<!--more-->

## The Problem

One You Lincolnshire is an NHS adult lifestyle service covering four programmes: Lose Weight, Smoke Free, Move More, and Drink Less. Clients self-refer online. The referral form is the front door to the entire service.

The challenge is that four programmes with overlapping eligibility criteria, different clinical questions, and different pathway logic do not fit neatly into a single linear form. A client interested in both losing weight and stopping smoking needs to go through smoking cessation eligibility questions and weight management questions, but in the right order, skipping irrelevant pages, with clinical scoring calculated along the way.

On top of that, postcode eligibility had to be checked in real time mid-form before a client got too far. And if a client dropped off halfway through, the organisation still needed to capture what data they had entered.

Dynamics 365 native forms couldn't handle any of this. I needed a different approach.

## Platform Choice

I use Plumsail Forms as the public-facing data capture layer for complex intake scenarios. It supports custom JavaScript, multi-page wizard layouts, real-time API calls mid-form, and direct Power Automate triggers on submission. It feeds cleanly into Dataverse on submit while handling things native forms simply cannot do at the front end.

The OYL form ended up at 14 pages, 109 unique fields, and four clinical pathways.

## Thinking Through the Architecture

The routing logic was the first thing to solve. Page one is a pathway selection. The client picks their primary interest. That single choice drives the routing for the entire rest of the form.

My approach was to track page visit state in JavaScript variables. Rather than routing clients down completely separate form branches, all pages exist in the wizard, but the routing logic controls the sequence and skips pages the client doesn't need. I built this using Plumsail's fd.container wizard on-change event, tracking early visit flags, page index sets, and skip lists.

Edge cases had to be handled explicitly. Pregnant clients skip the Lose Weight, Move More, and Drink Less pages regardless of their pathway selection. That's a clinical requirement, not a UI preference.

## The Azure Function

Real-time postcode eligibility checking was one of the more technically interesting parts.

When a client enters their postcode, the form calls an Azure Function that takes the postcode, looks it up against a Dataverse table of postcode areas, and returns whether they fall within the One You Lincolnshire service boundary. The client sees a Check my postcode button. All the API logic is invisible.

The form handles home postcode, work postcode, and study postcode. If the home postcode is ineligible, work and study questions are revealed and checked in turn. A valid GP surgery within the Lincolnshire boundary also makes a client eligible even if their home postcode falls outside the area. Both paths are evaluated independently and the most favourable result is used.

## Partial Save

On leaving the demographics page, a savePartialRecord() function fires once, guarded by a boolean flag to prevent double-firing. It collects the current form state via fd.data(), appends a randomly generated 99-character session code, and POSTs the payload to a Power Automate flow via an HTTP trigger URL.

That flow creates a partial lead record in Dataverse immediately, before the client has finished the form. If they drop off mid-way, the priority demographic data is already captured. The session code links that partial record to the eventual full submission, so when the client completes, the flow updates the existing record rather than creating a duplicate.

## Conditional Logic and Option Set Mapping

109 fields. All visibility and required state managed in JavaScript. Nothing conditionally shown server-side.

A representative example: the How did you hear about us field is a multi-select. Each selected option reveals a specific follow-up. Word of Mouth shows WhichMouth, Social Media shows WhichSocial, and so on for 11 different follow-up fields. Each is toggled by looping through a mapping object, converting selected values to strings, and comparing against Dataverse option set integers.

That last part matters. Dataverse option set fields store integers, not strings. ThriveTribe uses the 933960000-series convention. Eligibility flags across all four pathways are set in JavaScript as the client moves through the questions. By the time the submission payload reaches Power Automate, every value is already in the correct integer format for Dataverse.

Multi-select fields in Plumsail return arrays. They always need to be cast with Array.map(String) before comparison to prevent type mismatch failures. I learned that the hard way early on.

## Clinical Scoring

The Drink Less pathway includes the AUDIT questionnaire, a validated clinical alcohol risk assessment tool. I built the scoring logic entirely in JavaScript. Part 1 is seven questions scored 0 to 4 each. The Part 1 total determines whether Part 2 appears at all. A score below 5 causes the wizard to skip Part 2 entirely. Final scores populate clinical fields that are passed through to Dataverse on submission.

BMI is calculated in JavaScript from the client's height and weight, handling both metric and imperial inputs with unit conversion. The calculated value determines whether the Lose Weight pathway page appears at all.

## Submission

On submission, a Power Automate flow receives the full field payload as JSON and maps each field to the corresponding Dataverse lead column. Multi-select fields are serialised with a semicolon delimiter because that is what Dataverse multi-select option sets expect. The session code is used to check for an existing partial record and update it rather than creating a duplicate.

## Honest Limitations

The Power Automate flow URL is embedded in the client-side JavaScript. Anyone inspecting the page source can find the trigger endpoint. For a public form that is a real limitation. The mitigation is that the flow validates the payload structure before writing to Dataverse, so malformed requests are rejected. But ideally the trigger would sit behind an authenticated API layer. Worth acknowledging.

## The Result

A publicly accessible 14-page referral form handling four NHS programme pathways with real-time postcode eligibility checking, partial save on drop-off, clinical scoring, and clean Dataverse integration on submission. Non-technical staff don't see any of that complexity. They see a clean multi-step form that tells them if they're eligible and takes their referral.

That's the job.
