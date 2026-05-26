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

We had an issue that was affecting us for 2 years.

<!--more-->

Technically it wasn't my responsibility. It was an issue with our custom-built assessment website called Basecamp. We had a separate team of developers that worked on this, nothing to do with Power Platform. Except for where it tied in with data on Dynamics, as when a staff member completed a session with a client that data would land in the Dynamics database for reporting.

The crux of the issue was that the Service Bus had been getting overloaded. Due to the growth of the company there were more messages than it could handle, causing it to drop messages entirely, not just delay them. So our Cosmos DB would end up with incomplete referral records. Each client should be booked into 13 sessions, however we were seeing clients booked into only 9, or 10, or 5. It was quite random but it was a common issue. The workaround? Raise a ticket with our support team and they would manually go in and re-appoint them the missing session details. This was a long and tedious task, taking precious time away from actual tickets that needed looking at.

**I was called by the head of this service and asked if I could do some overtime and help reduce the number of lost bookings manually. That's when it clicked.**

I took a moment, then said: how about instead of manually fixing the data, I fix the whole thing.

They were shocked. Two years we'd had this issue, costing thousands per month in missed client data, and no one had found a solution. And here I come, from a different team entirely, saying I can fix the problem.

They had spent a lot of time and money trying to fix the Service Bus capacity issue but never found success with that as a solution. The most they could do was allow for parallel loading so it could process 2 messages at a time, in hopes it would reduce the amount of affected data. Which it did, for a while. But as the company kept growing and referrals kept increasing, the issue kept persisting.

## The Fix

I realised that within a service there would always be at least one referral that was accurate and complete, with all 13 sessions. So why not use that as a source of truth? Pull the missing data from there and reconstruct the incomplete referral that way.

And so that's exactly what I did.

It comprised of 2 flows in a solution.

**The detection flow.**

This flow runs daily. It queries Cosmos for referrals that are missing session data. When it finds affected records it triggers the fix flow for each one.

**The fix flow.**

Instead of reconstructing the data from scratch, the fix flow queries Cosmos for a reference client on the same service without any missing session data. It diffs the two booking arrays, identifies what session dates are missing, and borrows the structural data from the referenced client to construct the missing session objects.

The difficult part was ordering, since each session had to be in sequence from 1 to 13. So I built a SessionOrderMap as a Compose action, mapping each sessionType to the correct position in the sequence. The array lands clean and in the correct order.

On top of that the flow writes an audit record to Dataverse on every fix, and sends a team notification to operations after each run.

**What if staff want to fix a record themselves?**

I adapted a button that lives in the MDA whereby the advisor just has to select the broken record, press the button, and it will run the fix flow on that specific record immediately.

## The Result

When I presented this solution I got a flood of praise, and honestly that's the best part of this job. Seeing your work have a direct impact on people gives me a sense of fulfilment and pushes me to keep going every day.

The manual fix process is gone. The time our support staff spent fixing these records daily can now be spent elsehwere where it's needed. Daily detection means future affected records get caught and corrected automatically. The thousands lost per month, wiped. No more uncertainty with the system, no more frustration for advisors dealing with broken client records.

I could not fix the root cause. But our advisors no longer have to deal with missed client data, and neither does the business.

If you want me to explain this in greater detail just send me a message and I can do that no problem.

---
