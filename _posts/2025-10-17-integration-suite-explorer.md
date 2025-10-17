---
title: "Automatic AS-IS reports in SAP Integration Suite."
date: 2025-10-17
permalink: /posts/2025/10/integration-suite-explorer/
tags:
  - sap
  - integration
  - automation
  - python
  - docker
  - streamlit
---

When you work with SAP Integration Suite, you quickly realize that the hard part isn’t always building new interfaces. It’s understanding what’s already there. Over time, integration packages pile up, documentation gets stale, and nobody really knows the real, up-to-date state of the integration landscape.

I wanted a way to explore an existing landscape quickly, to generate a simple "as-is" report without manually opening each IFlow, which is a tedious, repetitive task. That’s how this little tool came to exist.

![Screenshot](https://raw.githubusercontent.com/malmriv/malmriv.github.io/refs/heads/master/images/integration-suite-explorer-demo.png?raw=true)

I decided to package it as a webtool. It takes a set of integration packages download as ZIPs, extracts their contents, analyses each flow, and produces a CSV report with each row listing the full details of each adapter: endpoints, protocols, parametrisations, internal dependences, etc.

This way, one gets a structured overview of what’s running in each tenant, without having to click through dozens of artifacts. The outline of how this is done, and some more technical details, can be found in this [GitHub](https://github.com/malmriv/integration-suite-explorer) repo.

---

## How it works

The app is written in Python, wrapped in Streamlit, and runs fine in a Docker container.  
At a high level, it does this:

1. You upload one or more integration package ZIPs.  
2. It unpacks them and finds all `.iflw` files.  
3. For each flow, it extracts:  
   - Adapter type and direction (sender or receiver)  
   - Transport protocol  
   - Endpoint address (and whether it’s parameterized)  
4. It also checks for internal dependencies between flows — when one IFlow calls another.  
5. Everything ends up in a CSV file that you can download and open in Excel.

That’s it. No database, no config files, no setup. Just upload, wait a few seconds, and you get your report.

---

## Internal calls and parameters

Integration flows often call each other using connectors known as ProcessDirect, which avoid performing an HTTP call over the internet just to end up in the same machine. The script maps these links, so the final CSV shows both "CallsIflow" and "IsCalledByIflow" columns. This proves especially valuable when adjusting the log level of several iFlows to Trace, since knowing the potential downstream effects of a single HTTP call allows the task to be completed much more efficiently.

The tool also checks whether adapter addresses are hardcoded or parameterized using the `{{parameter}}` pattern. If a default value exists in `parameters.prop`, it’s used in the output and the script marks the entry as parameterized. Knowing if an integration holds parametrized values is useful because these are bound to change between environments.

---

## Output

The generated CSV includes:

- `UID`, `Package`, `Iflow`, `IflowID`, `IflowVersion`  
- `AdapterType`, `TransportProtocol`, `AdapterDirection`, `AdapterAddress`  
- `IsParametrized`  
- `CallsIflow`, `IsCalledByIflow`

It’s enough to build a picture of how your tenant is wired: which flows talk to which, and where each adapter points to.

---

## Why this exists

I built this mostly because I got tired of doing repetitive “landscape discovery” tasks.  
People keep asking for an updated list of endpoints, or which IFlows are calling which others, and you end up manually opening a lot of integration flows and clicking through the UI.  

This script automates that. It it covers most cases, works fine in both Neo and Cloud Foundry-based subaccounts, and it’s easy to extend if you want to include other adapters or metadata.

If you want to try it, everything’s on [GitHub](https://github.com/malmriv/integration-suite-explorer). 
