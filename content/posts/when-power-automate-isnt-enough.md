---
title: "When Power Automate Isn't Enough, There Are Plugins"
date: 2026-05-23
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - power-automate
  - plugins
  - dataverse
  - dynamics-365
  - power-platform
draft: false
cover: /images/Plugin.jpg
---

Like most people, the first thing I did as a Power Platform developer was build a flow. Watched it run. That green tick, the satisfaction of knowing you've just saved someone time somewhere.

Since then I've built somewhere around 200 of them.

<!--more-->

I loved it. Automating processes, connecting systems, getting different platforms to speak to each other. I felt like a wizard.

Then I hit a wall.

Not a small wall. The kind that stops a project. I had a list of things I needed to build and no viable way to build them. Ownership assignment that had to happen before a record existed. An audit trail that needed to know what a field said before someone changed it. Validation logic that had to stop an operation completely, no partial saves, no workarounds. Flows couldn't do any of it. By the time a flow fires, the record is already saved. The transaction is closed. You're too late.

I had heard about plugins for years. Never looked seriously. Low code platform, you don't need to write code, that's the pitch. Turns out that's not entirely correct.

So I looked into it properly.

## What a Plugin Actually Is

A plugin is a C# assembly. It runs directly inside Dataverse, inside the transaction, before or after the database operation completes. Not outside it, not after it. Inside.

That distinction is everything.

Flows run outside Dataverse. Asynchronously. After the record saves. Fine for most things. A problem when timing matters, when you need to stop something before it happens, or capture something before it changes.

Three stages fire in sequence around every Dataverse operation. PreValidation, PreOperation, PostOperation. You choose where your plugin runs.

**PreValidation** runs before any transaction opens. Throw an error here and the operation stops completely. Clean. No side effects.

**PreOperation** runs inside the transaction, before the record saves. Modify the record here and those changes land in the database as if they were always there.

**PostOperation** runs inside the transaction, after the record saves. Create or update related records in response to what just happened.

A flow has none of this. It doesn't know these stages exist.

## What I Actually Built

Three things had been sitting on my backlog for months. No solution, just a note saying "blocked."

Automatic ownership assignment was the first. A lead hits the system and the correct team needs to own it immediately, not after a flow catches up, not with a gap where someone else might grab it. PreOperation. Ownership set before the record exists.

Outreach event logging was the one I was least sure how to solve. Every time a contact attempt field changes, we needed the old value, the new value, who changed it, and when. Flows genuinely cannot do this. By the time the trigger fires the old value is gone. Plugins have pre-images, a snapshot of the record before the operation. Compare old against new. Log the delta. Done.

The third was LTFU reactivation logic. Loss to follow-up cases reactivating under specific conditions. Timing-sensitive, transaction-dependent. I couldn't risk a flow firing late or out of order.

All three sat blocked for months. Plugins unblocked them in one week.

## Building Your First One

If you can build a flow you already understand the concept. Something happens, logic runs, data changes. The difference is where that logic lives.

### What you need

Visual Studio — community edition is free. The Dataverse SDK, which you install via NuGet after creating your project (search for `Microsoft.CrmSdk.CoreAssemblies`). And the Plugin Registration Tool. If you have PAC CLI installed, run `pac tool prt` in the terminal. You should have PAC CLI.

### Setting up the project

Create a new Class Library project in Visual Studio. When it asks for the framework, choose **.NET Framework 4.6.2**. This matters. Dataverse does not support .NET Core or anything newer for plugins. Get it wrong and your assembly won't load. Once the project is created, right-click Dependencies, open Manage NuGet Packages, and install the SDK.

### Writing the plugin

Every plugin is a C# class that implements `IPlugin`. At the start of your execute method you need two things:

```csharp
IPluginExecutionContext context = (IPluginExecutionContext)serviceProvider
    .GetService(typeof(IPluginExecutionContext));

IOrganizationServiceFactory factory = (IOrganizationServiceFactory)serviceProvider
    .GetService(typeof(IOrganizationServiceFactory));

IOrganizationService service = factory.CreateOrganizationService(context.UserId);
```

`context` is your trigger. It tells you what happened, which record, and which stage you're at. `service` is your Dataverse connection. Read and write records with it, same as Dataverse actions inside a flow.

Pull the target record from context:

```csharp
Entity targetEntity = (Entity)context.InputParameters["Target"];
```

From there, read fields, check which message fired, write your logic.

### Pre-images

If you need to know what a field contained before an update, register a pre-image when you set up your step. It arrives like this:

```csharp
Entity preImage = context.PreEntityImages["PreImage"];
```

Compare old against new. Act only when something changed. This is how the outreach logging works — without the pre-image, the old value is already gone by the time the plugin runs.

### Registering and deploying

Build the project. You get a `.dll` in your bin folder. Open the Plugin Registration Tool, connect to your environment, register the assembly. Then register a step: the message (Create, Update, Delete, Assign), the table, the stage, and optionally filtering attributes so it only fires on specific field changes.

Add the assembly to a Dataverse solution and let your ALM pipeline handle promotion through DEV, UAT, and PROD. No manual re-registration in each environment. It travels with the solution.

## When to Use One

Plugins are not a replacement for flows. They're the right tool for a specific set of problems.

Logic that must run before the record saves. Data you need to capture before it changes. Failures that need to roll back the entire operation. Timing that a flow cannot guarantee.

Everything else, build a flow.

The plugin doesn't make you a better developer because it's more complex. It makes you a better developer because you stopped reaching for the wrong tool.
