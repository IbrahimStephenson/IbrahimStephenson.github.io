---
title: "How I Fixed a Two-Year Data Bug That Was Costing Thousands Every Month"
date: 2026-04-04
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
cover: /images/datafix.png
---

Some bugs sit in the backlog for months. This one had been sitting there for two years!

<!--more-->

## Discovering the Problem

When I started digging into our data pipelines, one thing kept coming up in conversations with the operations team. Clients on NHS programmes were failing to enrol properly, and advisors were spending 15 to 30 minutes manually fixing each one. Over 60 clients a month. The maths on that is grim.

The setup was this: Microsoft Bookings handled appointment scheduling. Session data got passed from Bookings into Azure Cosmos DB via an Azure Service Bus. That Cosmos data was what everything downstream used to track a client's programme journey.

The problem was that the Service Bus had been configured without adequate capacity. Under load it was dropping messages. Not delaying them. Dropping them. So Cosmos DB would end up with incomplete referral records. A client might have nine sessions written when they needed all thirteen to complete their NHS programme pathway. Incomplete data meant failed enrolment. Failed enrolment meant the organisation was not getting paid for that referral.

Two years. Thousands of pounds a month. Ongoing leadership escalations. And the fix was 15 to 30 minutes of manual advisor time per client, every single time.

## Thinking Through the Options

The obvious fix would have been the root cause: the Service Bus capacity. But that sat within a third-party integration layer outside my remit. I could not touch it.

So the question became: given that messages are going to keep being dropped, how do I detect when it happens and correct the data automatically?

My first instinct was to reconstruct the missing sessions from scratch. Hardcode the expected session types and values, fill the gaps. I ruled that out quickly. Session structures vary between services and programmes. Hardcoding would have been brittle and needed constant maintenance as things evolved.

Then I thought about it differently. If a referral is incomplete, there will almost certainly be another referral on the same service that is complete. Why not use that as a source of truth?

## The Solution

I built two Power Automate flows packaged as a managed Dataverse solution.

### Detection flow

The detection flow runs on a daily schedule. It queries Cosmos for referrals in FirstSession status with fewer than 14 booking records. All NHS programme referrals should have exactly 13 sessions plus an initial assessment, so anything below that threshold gets flagged. When it finds affected records it triggers the fix flow for each one.

### Fix flow

Rather than reconstructing sessions from scratch, the fix flow queries Cosmos for a reference client on the same service with a full set of 13 sessions. It diffs the two bookings arrays, identifies exactly which session types are missing, borrows the structural data from the reference client, and constructs the missing session objects from that.

The tricky part was ordering. Cosmos does not guarantee array order, but the downstream flows expected sessions in programme sequence. Just merging the arrays would append missing sessions at the end and break everything downstream.

I built a SessionOrderMap as a Compose action, a static key-value object mapping each sessionType to its correct position in the sequence. Tagged each booking with a sort index, ran the sort function on the tagged array, then stripped the temporary index before writing back to Cosmos. The array lands clean, correctly ordered, ready for whatever comes next.

On top of that the flow writes an audit record to Dataverse on every fix, a permanent log of what was corrected and when. A Teams notification goes to the operations team after each run. Dataverse session lookup fields get updated to keep the CRM in sync.

## The Result

185 backlogged referrals cleared on the first run.

The manual 15 to 30 minutes per client is gone. Daily detection means future affected records get caught and fixed automatically. Leadership escalations stopped. The operations team gets a Teams notification every morning so they have full visibility without ever needing to open Cosmos.

The Service Bus root cause is still there. The right long-term fix is increasing throughput tiers or implementing dead-letter queue handling so messages are retained on failure rather than dropped. But the data is clean, the revenue is protected, and the advisors got their time back.
