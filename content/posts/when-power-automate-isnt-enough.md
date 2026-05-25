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

Like most people, the first thing I did as a power platform developer was create a power automate flow. It felt amazing, seeing it run in action. That green tick, the satisfaction knowing you've just saved someone time somewhere.

And since then I have built somewhere around 200 flows.

<!--more-->

I loved how I was able to automate processes, connect systems, utilise custom connectors and get different platforms to speak to each other. I felt like a wizard.

But eventually I hit a wall. I've discovered the limitations, the bottlenecks. I realised although flows are GREAT for a lot of things, they cannot do everything. So I had to look elsewhere.

Then I learnt about plugins. I've heard about plugins for a long time, never delved deeper than just mild curiosity. So I decided to do some proper research, look into it seriously. Power platform is promoted as a low code platform, you can do everything you need without writing a line of code. Well that's not entirely correct.

A plugin is a C# assembly, it triggers directly inside Dataverse at different stages, pre validation, pre operation or post operation. It's a server side event handler that runs in the platform itself.

Using custom plugins I was able to finally complete some of the dev work I've previously had to put off as I did not have a viable solution for it, and also improve some of my previous work by utilising these plugins for those use cases as well. Automatic Ownership assignment, Outreach Event Logging, LTFU reactivation logic.

If you're reading this post and, like the old me, have never looked into plugins seriously, then take this as your wake up call. Check them out and you'll be surprised at how much utilisation you can get out of them.

## Creating and Deploying a Plugin

If you can build a flow you already understand the concept. Something happens, logic runs, data changes. Plugins work the same way. The difference is where and when that logic executes. Flows run outside Dataverse, asynchronously, after the record is already saved. Plugins run inside the platform itself, inside the transaction, before or after the database operation completes. That distinction is everything.

### What You Need Before You Start

Three things.

**Visual Studio** — Community edition is free. Get it from the Microsoft website if you don't have it already.

**The Dataverse SDK** — Once your project is created, install this via NuGet. Search for `Microsoft.CrmSdk.CoreAssemblies`. It gives you everything you need to talk to Dataverse from C#.

**Plugin Registration Tool** — How you deploy your plugin to an environment. Run `pac tool prt` in the terminal if you have PAC CLI installed, which you should if you're doing any serious Power Platform work.

### Setting Up Your Project

Open Visual Studio and create a new project. Search for Class Library and select it. When it asks you which framework, choose **.NET Framework 4.6.2**. This part matters. Dataverse does not support .NET Core or anything newer than that for plugins. Get this wrong and your assembly will not load.

Once the project is created, right click Dependencies in the solution explorer, click Manage NuGet Packages, search for `Microsoft.CrmSdk.CoreAssemblies` and install it. You are now ready to write your first plugin.

### Writing the Plugin

Every plugin is a C# class that implements `IPlugin`. If you are using a shared PluginBase class like we do, your plugin just inherits from that and overrides `ExecuteCrmPlugin`. Either way the shape is the same.

The two things you always need at the start of your execute method are the context and the service:

```csharp
IPluginExecutionContext context = (IPluginExecutionContext)serviceProvider
    .GetService(typeof(IPluginExecutionContext));

IOrganizationServiceFactory factory = (IOrganizationServiceFactory)serviceProvider
    .GetService(typeof(IOrganizationServiceFactory));

IOrganizationService service = factory.CreateOrganizationService(context.UserId);
```

Think of `context` as your trigger. It tells you what happened, which record was affected, and what stage you are at. Think of `service` as your Dataverse connector. It is how you read and write records, the same as Dataverse actions inside a flow.

From the context you can pull the target record like this:

```csharp
Entity targetEntity = (Entity)context.InputParameters["Target"];
```

From there you can read fields, check which message fired, check which stage you are at, and write your logic.

### Understanding the Pipeline

There's no equivalent to this in flows, so it's worth understanding before you write anything.

When something happens in Dataverse, three stages fire in sequence before and after the database operation. You choose which one your plugin runs at.

**PreValidation** runs before any transaction opens. Use this if you want to throw a validation error and stop the operation completely, cleanly, with no side effects.

**PreOperation** runs inside the transaction, before the record saves. This is where you modify the record before it hits the database. In the LeadPlugin, ownership gets set here so the correct team is assigned before the lead even exists in the system.

**PostOperation** runs inside the transaction, after the record saves. This is where you create or update related records in response to what just happened.

A flow has none of this. By the time your flow trigger fires, the record is already saved and the transaction is long closed. Fine for most things. A problem when timing matters.

### Pre-Images: Knowing What Changed

If you need to know what a field contained before an update, you need a pre-image. Flows genuinely cannot do this.

When you register your plugin step you can attach a pre-image, a snapshot of the record as it was before the operation. It arrives in your plugin like this:

```csharp
Entity preImage = context.PreEntityImages["PreImage"];
```

You can then compare the old value against the new one and only act when something actually changed. The contact outreach audit trail at ThriveTribe works exactly this way. Every time the contact attempt field changes, we capture the old value, the new value, who changed it, and when. Without the pre-image that is not possible.

### Registering Your Plugin

Once your code is written, build the project. You will get a `.dll` file in your bin folder. This is what you deploy.

Open the Plugin Registration Tool and connect to your environment. Click Register New Assembly, browse to your DLL, and upload it. Dataverse now knows your plugin exists.

Next you register a step. This is the equivalent of configuring a flow trigger. You specify the message (Create, Update, Delete, Assign), the table it fires on, the stage, and optionally a list of filtering attributes so the plugin only fires when specific fields change rather than on every single update.

If you need a pre-image, right click the step once it is created and add a pre-image from there. Give it the alias `PreImage` and it will arrive in your code exactly as shown above.

### Deploying Across Environments

Manually uploading a DLL to each environment is not a real deployment process. Add your plugin assembly to your Dataverse solution and let your ALM pipeline handle promotion through DEV, UAT, and PROD. The DLL gets packaged into the solution zip like any other component — no manual re-registration needed in each environment.

If you have not set up an ALM pipeline yet, I covered that in a previous post.

### Knowing When to Use One

Plugins are not a replacement for flows. They are the right tool for a specific set of problems.

If the logic needs to run before the record saves, use a plugin. If you need to know what a field was before the change, use a plugin. If a failure needs to roll back the entire operation, or the timing matters in a way a flow can't guarantee, use a plugin.

Otherwise, build a flow.
