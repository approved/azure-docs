---
title: Migrate to the Azure Monitor agent (AMA) from the Log Analytics agent (MMA/OMS) for Microsoft Sentinel
description: Learn about migrating from the Log Analytics agent (MMA/OMS) to the Azure Monitor agent (AMA), when working with Microsoft Sentinel.
author: batamig
ms.topic: reference
ms.date: 02/09/2022
ms.author: bagol
---

# AMA migration for Microsoft Sentinel
This article describes the migration process to the Azure Monitor Agent (AMA) when you have an existing Log Analytics Agent (MMA/OMS), and are working with Microsoft Sentinel. 

> [!IMPORTANT]
> The Log Analytics agent will be [retired on **31 August, 2024**](https://azure.microsoft.com/updates/were-retiring-the-log-analytics-agent-in-azure-monitor-on-31-august-2024/). If you are using the Log Analytics agent in your Microsoft Sentinel deployment, we recommend that you start planning your migration to the AMA.

## Prerequisites
Start with the [Azure Monitor documentation](/azure/azure-monitor/agents/azure-monitor-agent-migration) which provides an agent comparison and general information for this migration process. 

This article provides specific details and differences for Microsoft Sentinel.


## Gap analysis between agents
The following tables show gap analyses for the log types that currently rely on agent-based data collection for Microsoft Sentinel. This will be updated as support for AMA grows towards parity with the Log Analytics agent. 

> [!IMPORTANT]
> The AMA currently has a limit of 5,000 Events Per Second (EPS). Verify whether this limit works for your organization, especially if you are using your servers as log forwarders, such as for Windows forwarded events or Syslog events.

### Windows logs

|Log type / Support  |Azure Monitor agent support |Log Analytics agent support  |
|---------|---------|---------|
|**Security Events**     |  [Windows Security Events data connector](data-connectors-reference.md#windows-security-events-via-ama)  (Public preview)     |  [Windows Security Events data connector (Legacy)](data-connectors-reference.md#security-events-via-legacy-agent-windows)       |
|**Filtering by security event ID**     |   [Windows Security Events data connector (AMA)](data-connectors-reference.md#windows-security-events-via-ama)  (Public preview)    |     -     |
|**Filtering by event ID**     | Collection only        |   -       |
|**Windows Event Forwarding**     |  [Windows Forwarded Events](data-connectors-reference.md#windows-forwarded-events-preview) (Public Preview)       |     -     |
|**Windows Firewall Logs**     |  -        |  [Windows Firewall data connector](data-connectors-reference.md#windows-firewall)       |
|**Performance counters**     |   Collection only      |  Collection only       |
|**Windows Event Logs**     |  Collection only       | Collection only        |
|**Custom logs**     |   -       |    Collection only     |
|**IIS logs**     |    -      |    Collection only     |
|**Multi-homing**     |  Collection only       |   Collection only      |
|**Application and service logs**     |    -      |    Collection only     |
|**Sysmon**     |    Collection only      |      Collection only   |
|**DNS logs**     |   -       | Collection only        |


### Linux logs

|Log type / Support  |Azure Monitor agent support |Log Analytics agent support  |
|---------|---------|---------|
|**Syslog**     |  Collection only      |   [Syslog data connector](connect-syslog.md)      |
|**Common Event Format (CEF)**     |  Collection only       |  [CEF data connector](connect-common-event-format.md)       |
|**Sysmon**     |   Collection only    |  Collection only      |
|**Custom logs**     |   -       |  Collection only       |
|**Multi-homing**     |   Collection only      |     -     |


## Recommended migration plan

Each organization will have different metrics of success and internal migration processes. This section provides suggested guidance to considered when migrating from the Log Analytics MMA/OMS agent to the AMA, specifically for Microsoft Sentinel.

**Include the following steps in your migration process**:

1. Make sure that you've considered your environmental requirements and understand the gaps between the different agents. For more information, see [Plan your migration](../azure-monitor/agents/azure-monitor-agent-migration.md#plan-your-migration) in the Azure Monitor documentation.

1. Run a proof of concept to test how the AMA sends data to Microsoft Sentinel, ideally in a development or sandbox environment.

    1. To connect your Windows machines to the [Windows Security Event connector](data-connectors-reference.md#windows-security-events-via-ama), start with **Windows Security Events via AMA** data connector page in Microsoft Sentinel. For more information, see [Windows agent-based connections](connect-azure-windows-microsoft-services.md#windows-agent-based-connections).

    1. Go to the **Security Events via Legacy Agent** data connector page. On the **Instructions** tab, under **Configuration** > Step 2, **Select which events to stream**, select **None**. This configures your system so that you won't receive any security events through the MMA/OMS, but other data sources relying on this agent will continue to work. This step affects all machines reporting to your current Log Analytics workspace.

    > [!IMPORTANT]
    > Ingesting data from the same source using two different types of agents will result in double ingestion charges and duplicate events in the Microsoft Sentinel workspace. 
    >
    > If you need to keep both data connectors running simultaneously, we recommend that you do so only for a limited time for a benchmarking, or test comparison activity, ideally in a separate test workspace.
    >

1. Measure the success of your proof of concept. 

    To help with this step, use the **AMA migration tracker** workbook, which displays the servers reporting to your workspaces, and whether they have the legacy MMA, the AMA, or both agents installed. You can also use this workbook to view the DCRs collecting events from your machines, and which events they are collecting.

    For example:

    :::image type="content" source="media/ama-migrate/migrate-workbook.png" alt-text="Screenshot of the AMA migration tracker workbook." lightbox="media/ama-migrate/migrate-workbook.png" :::

    Success criteria should include a statistical analysis and comparison of the quantitative data ingested by the MMA/OMS and AMA agents on the same host:

    - Measure your success over a predefined time period that represents a normal workload for your environment.

    - While testing, make sure to test each new feature provided by the AMA, such as Linux multi-homing, Windows event filtering, and so on.

    - Plan your rollout for AMA agents in your production environment according to your organization's risk profile and change processes.

3. Roll out the new agent on your production environment and run a final test of the AMA functionality.

4. Disconnect any data connectors that rely on the legacy connector, such as Security Events with MMA. Leave the new connector, such as Windows Security Events with AMA, running.

    While you can have both the legacy MMA/OMS and the AMA agents running in parallel, prevent duplicate costs and data by making sure that each data source uses only one agent to send data to Microsoft Sentinel.

5. Check your Microsoft Sentinel workspace to make sure that all your data streams have been replaced using the new AMA-based connectors.

6. Uninstall the legacy agent. For more information, see [Manage the Azure Log Analytics agent ](/azure/azure-monitor/agents/agent-manage#uninstall-agent).

## FAQs
The following FAQs address issues specific to AMA migration with Microsoft Sentinel. For more information, see also the [Frequently asked questions for AMA migration](/azure/azure-monitor/faq#azure-monitor-agent) in the Azure Monitor documentation.

## What happens if I run both MMA/OMS and AMA in parallel in my Microsoft Sentinel deployment?
Both the AMA and MMA/OMS agents can co-exist on the same machine. If they both send data, from the same data source to a Microsoft Sentinel workspace, at the same time, from a single host, duplicate events and double ingestion charges will occur.

For your production rollout, we recommend that you configure either an MMA/OMS agent or the AMA for each data source. To address any issues for duplication, see the relevant FAQs in the [Azure Monitor documentation](/azure/azure-monitor/faq#azure-monitor-agent).

## The AMA doesn’t yet have the features my Microsoft Sentinel deployment needs to work. Should I migrate yet?
The legacy Log Analytics agent will be retired on 31 August 2024.

We recommend that you keep up to date with the new features being released for the AMA over time, as it reaches towards parity with the MMA/OMS. Aim to migrate as soon as the features you need to run your Microsoft Sentinel deployment are available in the AMA.

While you can run the MMA and AMA simultaneously, you may want to migrate each connector, one at a time, while running both agents.



## Next steps

For more information, see:

- [Frequently asked questions for AMA migration](/azure/azure-monitor/faq#azure-monitor-agent)
- [Overview of the Azure Monitor agents](/azure/azure-monitor/agents/agents-overview)
- [Migrate from Log Analytics agents](/azure/azure-monitor/agents/azure-monitor-agent-migration)
- [Windows Security Events via AMA](data-connectors-reference.md#windows-security-events-via-ama)
- [Security events via Legacy Agent (Windows)](data-connectors-reference.md#security-events-via-legacy-agent-windows)
- [Windows agent-based connections](connect-azure-windows-microsoft-services.md#windows-agent-based-connections)
