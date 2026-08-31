# Standing Up a Microsoft Sentinel Environment for SC-200

**Date:** August 31, 2026
**Platform:** Microsoft Sentinel / Azure
**Domain:** SC-200 - Manage a Security Operations Environment

---

## What I Built

- Created a dedicated resource group for the Sentinel platform itself (workspace, Sentinel instance, analytics rules), kept separate from resource groups holding actual machine inventory
- Deployed a Log Analytics workspace and enabled Microsoft Sentinel on top of it
- Assigned myself the Microsoft Sentinel Contributor role instead of relying on subscription Owner, to practice the kind of least-privilege access a real analyst account would actually get

---

## Data Connectors Configured

- **Windows Security Events via AMA** - connected an Azure-hosted Windows VM, created a Data Collection Rule targeting it, confirmed events flowing with `SecurityEvent | take 10` in the Logs blade
- **Microsoft Entra ID** - sign-in and audit log ingestion
- **Azure Activity** - subscription-level activity log ingestion
- **Windows Firewall Events** - via AMA
- **Windows Forwarded Events (WEF)** - via AMA
- **Syslog and CEF (in progress)** - working through Linux log collection for my home lab VMs, which needs Azure Arc onboarding first since those machines live outside Azure entirely and Sentinel can't target a Data Collection Rule at a resource it doesn't know about


---

## Custom Tables and Resource Organization

- Built a custom table in the workspace to store data outside the default schema
- Learned to keep Arc-connected on-prem machine inventory in its own resource group, separate from both the Sentinel platform RG and auto-generated ones like a VM's own network resource bundle or NetworkWatcherRG. Keeping "the monitoring platform" and "the things being monitored" as separate resource groups makes either one manageable on its own, which matters more once you're thinking about how a real MSSP organizes client environments

---

## Monitoring Ingestion

- Used workbooks to look at ingestion volume and cost per connector, to start building the habit of watching what's actually expensive to ingest instead of just switching every connector on and finding out later

---

## What I Learned Today

**Big Picture:** A SIEM is only as good as its ingestion pipeline, and getting data connectors right (native cloud connectors vs. agent-based AMA vs. Arc-required hybrid onboarding) is its own skill, separate from writing detection logic on top of the data once it's there.

- Data connectors split into two real categories: native connectors that just need permissions (Entra ID, Azure Activity), and agent-based connectors that need a Data Collection Rule pointed at either a native Azure resource or an Arc-registered one
- Resource group naming isn't just tidiness. Separating the monitoring platform from the machine inventory it watches means either one can be managed or handed off without touching the other
