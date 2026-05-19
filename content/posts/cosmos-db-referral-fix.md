---
title: "How I Fixed a Two-Year Data Bug That Was Costing Thousands Every Month"
date: 2026-05-20
author: Ibrahim Stephenson
categories:
  - Power Platform
  - Azure
tags:
  - power-automate
  - cosmos-db
  - azure-service-bus
  - dataverse
draft: false
---

Some bugs sit in the backlog for months. This one had been sitting there for two years.

<!--more-->

## Discovering the Problem

When I started digging into ThriveTribe's data pipelines, one issue kept coming up in conversations with the operations team. Clients on NHS programmes were failing to enrol properly, and advisors were spending 15 to 30 minutes manually fixing each one. Over 60 clients a month. The maths on that is grim.

The setup was this: Microsoft Bookings handled appointment scheduling. Session data from Bookings was passed into Azure Cosmos DB via an Azure Service Bus. That Cosmos data was what downstream flows used to track a client's programme journey.

The problem was that the Service Bus had been configured without adequate capacity. Under load, it was dropping messages entirely. Not delaying them. Dropping them. So Cosmos DB would end up with incomplete referral records. A client might have nine sessions written when they needed all thirteen to complete their NHS programme pathway. Incomplete data meant failed enrolment. Failed enrolment meant the organisation wasn't getting paid for that referral.

Two years. Thousands of pounds a month. Ongoing leadership escalations. And the fix was 15 to 30 minutes of manual advisor time per client, every single time.

## Thinking Through the Options

The obvious answer would have been to fix the root cause: the Service Bus capacity. But that sat within Basecamp's integration layer, involving third-party licensing and infrastructure outside my remit. I couldn't touch it.

So the question became: given that messages are going to keep being dropped, how do I detect when it happens and correct the data automatically?

My first instinct was to reconstruct the missing sessions from scratch by hardcoding the expected session types and values. I quickly ruled that out. Session structures vary slightly between services and programmes. Hardcoding would have been brittle and would have needed constant maintenance as services evolved.

Then I thought about it differently. If a referral is incomplete, there will almost certainly be another referral on the same service that is complete. Why not use that as a source of truth?

That was the insight that made the whole solution work.

## The Solution

I built two Power Automate flows, packaged as a managed Dataverse solution called BasecampMissingDataSolution.

**Detection Flow**

This runs on a daily schedule. It queries the bcReferrals Cosmos container for referrals in FirstSession status with fewer than 14 booking records. All NHS programme referrals should have exactly 13 sessions plus an initial assessment, so anything below that threshold is flagged. When affected records are found, it triggers the fix flow for each one.

**Fix Flow**

This is where the interesting stuff happens. 51 actions, using Dataverse, Azure Cosmos DB, and Microsoft Teams connections.

Rather than reconstructing missing sessions from scratch, the flow queries Cosmos for a reference client on the same service with all 13 sessions present. It then diffs the two bookings arrays to identify exactly which session types are missing. It borrows the structural data from the reference client's matching sessions and constructs the missing session objects from that.

The tricky bit was ordering. Cosmos DB does not guarantee array order, but the downstream flows expected sessions in programme sequence from InitialAssessment through to Thirteenth. Just merging the arrays would append missing sessions at the end, breaking everything downstream.

I built a SessionOrderMap as a Compose action, a static key-value object mapping each sessionType string to its correct ordinal integer. I used addProperty to tag each booking with a sortIndex, ran the sort() function on the tagged array, then stripped the temporary sortIndex before patching back to Cosmos. The array lands clean, correctly ordered, ready for downstream processing.

On top of that, the flow writes an audit record to Dataverse on every fix so there is a permanent record of what was corrected and when. A Teams notification goes to the operations team after each run. Dataverse contact session lookup fields are updated to keep the CRM in sync.

## The Result

185 backlogged referrals cleared on the first run.

The manual 15 to 30 minute per-client workaround is gone. Daily detection means future affected records get caught and fixed automatically. Leadership escalations have stopped. The operations team gets a Teams notification every morning so they have full visibility without ever needing to open Cosmos directly.

The Service Bus root cause is still there. Eventually the right fix is increasing throughput tiers or implementing dead-letter queue handling so messages are retained on failure rather than dropped. But the data is clean, the revenue is protected, and the advisors got their time back.
