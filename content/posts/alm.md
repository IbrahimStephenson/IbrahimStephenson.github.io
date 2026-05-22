---
title: "How I Built a Proper ALM Pipeline for Power Platform (Starting From Chaos)"
date: 2026-05-22
author: Ibrahim Stephenson
categories:
  - Power Platform
tags:
  - alm
  - azure-devops
  - pac-cli
  - dataverse
  - power-automate
draft: false
cover: /images/ALM.png
---

Three environments. No pipeline. Everything unmanaged. DEV, UAT, and PROD all existed, but nobody could tell you with confidence what was in each one or how they had diverged. And it had been that way for years.

This is how I fixed it, and how painful that process actually was.

<!--more-->

## What Unmanaged Actually Means

If you have not dealt with this before, here is the problem with an unmanaged Power Platform environment.

In an unmanaged solution, every component is editable in every environment. Someone can open PROD, tweak a flow, save it, and nobody knows it happened. There is no deployment process. There is no audit trail. DEV, UAT, and PROD drift apart gradually until they are three different systems pretending to be one.

When I started properly investigating ThriveTribe's environments, there was a 5,634 component gap between DEV and PROD. Not a typo. Over five thousand components that existed in one place and not the other, with no record of how or when they diverged. Manually reconciling that was never going to work. PROD became the source of truth by necessity, because it was the only environment we knew was definitely correct.

## The Plan

The goal was a full managed solution pipeline. DEV stays unmanaged so developers can work freely. UAT and PROD get managed solutions only, locked down, no direct edits. Every change goes through the pipeline. That is the standard. That is what I was building toward.

The migration itself was a seven-phase operation planned across two nights. Phase zero was backups of everything before touching anything. Then consolidating PROD into a single exportable solution, getting DEV and UAT to parity, deploying managed to UAT, deploying managed to PROD, and finally reimporting any in-progress DEV work that had been saved aside.

The plan was solid. The execution was a different story.

## Where It Started Going Wrong

The first serious blocker was a third-party PCF control called IKL TextSMS4Dynamics. It had been installed at some point, used, and then effectively abandoned. But it was still baked into the solution XML. Every import attempt into UAT failed because the IKL dependency was declared in the solution but the IKL managed solution was not present in the target environment.

So I opened the solution ZIP and started doing XML surgery. Grepped through customizations.xml and solution.xml hunting for every reference to IKL, TextSMS, LiveChat. Removed the PCF control declarations. Removed the workflow registrations. Repackaged. Tried again.

Still failing. Because canvas app files are also ZIPs inside the main ZIP, and the binary grep had flagged false positives on them while the real references were hiding in the XML. Round two of surgery. Eventually got it clean.

Then a completely different error appeared.

## The DateTime Behaviour Problem

This one genuinely surprised me. The import failed because a field called ttd_startdate had been defined in the solution XML as UserLocal datetime, but the target environment already had it as DateOnly. A formula column called ttd_joinabledate had been built against the DateOnly version and was blocking the import from changing the field behaviour.

Dataverse does not let you change DateTime behaviour on a column that already has dependents through the normal UI. It just refuses. The only way through was more XML surgery: find every instance of ttd_startdate across multiple tables in customizations.xml, correct the Behavior values to match what the environment expected, add the CanChangeDateTimeBehavior flag to prevent future conflicts, repackage.

That worked. Then I got a 403 on a Cosmos DB connection reference.

## Connection References and the Settings File

Connection references are one of the most annoying parts of Power Platform ALM. Each environment has its own connections and the solution cannot hardcode them. The standard approach is a deploymentSettings.json file that maps each connection reference in the solution to the correct connection ID in the target environment.

I generated the settings file, mapped everything across, ran the import. The Cosmos DB connection kept throwing a permissions error because the connection in UAT was owned by a personal account rather than a service account. The import pipeline was running as a service principal that did not have access to someone's personal Cosmos connection.

The fix was consolidating connection references across the environment. Because doing it flow by flow manually was not realistic given the volume of flows, I built a PowerShell script using a dedicated Azure app registration. Even that took several attempts: smart quote encoding errors in the script file, a missing AcceptLicense parameter, an AADSTS error when the Azure CLI client ID was blocked by the tenant. The resolution was creating a dedicated app registration named ThriveTribe-PowerShell-Script with a native client redirect URI and Dynamics CRM delegated permission.

## The Layered Architecture

While all of this was happening I also redesigned the solution structure itself. The monolithic single-solution approach was part of why things had gotten messy. One solution containing everything means a failed import rolls back everything.

The new architecture is four layers. Layer zero is Components: plugins and PCF custom controls that everything else depends on. Layer one is Data: tables, columns, relationships, views, forms, option sets. Layer two is Flows: Power Automate flows and environment variables. Layer three is UI: canvas apps, model-driven apps, web resources.

The reason for the split is practical. If I deploy a flow change I should not have to touch the UI layer. If a UI deployment fails it should not roll back unrelated data changes. Each layer has its own solution, its own versioning, its own independent deployment sequence.

## The Azure DevOps Pipeline

Once the environments were stable I built the actual CI/CD pipeline in Azure DevOps.

Two YAML pipelines. The export pipeline runs against DEV: it installs PAC CLI, exports the unpacked solution XML, and commits it to Azure Repos using a System.AccessToken. The deployment pipeline takes that committed XML, packs it as managed, and imports it to UAT with force overwrite.

The Entra ID app registration for the service connection is named ThriveTribe-DevOps-SP. Two Power Platform service connections in Azure DevOps, one for DEV and one for UAT. Connection references are handled via deploymentSettings.json so nothing is hardcoded to environment credentials. Environment variables handle any configuration that differs between environments.

The result is that managed solutions can only reach UAT or PROD through the pipeline. No manual imports. No one making changes directly in production at midnight and hoping for the best.

## What It Actually Took

I am not going to pretend this was clean. It took multiple sessions of XML surgery, several rounds of debugging errors I had never seen before, a custom PowerShell script with its own debugging journey, and more ZIP file repacking than I care to count.

But the before and after is stark. Before: three environments drifting apart with no audit trail and no way to know what was actually in PROD. After: a layered architecture, a proper deployment pipeline, connection references consolidated, and every change going through version control before it touches a managed environment.

If you are building on Power Platform at any serious scale and you are still working unmanaged, this is the migration worth doing. Just set aside more time than you think you need.

---

*Built with PAC CLI, Azure DevOps YAML pipelines, Entra ID app registration, and far too many hours in a text editor looking at solution XML.*