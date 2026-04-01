---
description: Standard operating procedure (Workflow) for interacting with, building, and updating Apache NiFi flows
---

# NiFi Development Workflow

This workflow defines the standard steps for the Agent to interact with Apache NiFi via the MCP Server safely, accurately, and in strict adherence to the user's existing system conventions. This workflow should be automatically triggered whenever the user requests operations on NiFi flows.

## Step 1: Discovery & Understand
- Utilize the `search_nifi_flow` or `get_flow_outline` tools to accurately locate the target **Process Group** based on its Name or UUID as requested by the user.
- Use `document_nifi_flow` to extract and inspect the entire internal architecture (including Processors, Connections, Ports, Controller Services, and **specifically cross-check any Variables defined at the Process Group level**).
- Analyze and summarize the **business logic**, property configurations, **Variables list**, and Expression Language (EL) statements to gain a complete understanding of the flow's purpose.

## Step 2: Convention Analysis & Environment
- Extract naming and architectural conventions used within the Process Group (e.g., Processor naming conventions, Auto-Terminate configurations for unused Connections, usage of Funnels, Payload structures, Auth Contexts, Retry configurations).
- Extract environmental configurations (URL Patterns, standard API path structures, and **globally declared Variables**).
- Cross-reference these conventions and Variables to serve as the design foundation for building or updating the flow.

## Step 3: Implementation
- Formulate a detailed Implementation Plan before calling any flow creation API. Ensure you list **any required Variables** that must be assigned or inherited by the target Process Group to prevent Expression Language evaluation errors.
- Consider **taking a backup snapshot** of the original architecture if performing large-scale updates or deletions.
- Execute flow creation tools (`create_nifi_process_group`, `create_nifi_processors`, `create_nifi_connections`, etc.) to build new flows or update existing ones in **strict adherence** to the conventions analyzed in Step 2.
- A **custom approach** (creating net-new structures/methodologies) should only be applied if and only if the current system lacks any existing conventions to handle the user's specific use case.

## Step 4: Auto-Layout
// turbo
- Execute the `layout_nifi_process_group` tool immediately after successfully creating, copying, or connecting components. This step is strictly required to ensure the NiFi Canvas does not have overlapping components, keeping the UI clean, structured, and easy to maintain for operational engineers.

## Step 5: Validation & Debugging
- Use `get_process_group_status` or `analyze_nifi_processor_errors` to check the Health status and queue size of the flow immediately after creation or modification.
- Ensure all processors are in a `VALID` state. Specifically, **cross-check Process Group Variables** to confirm no Expression Language references an undefined variable (which would trigger a missing variable warning).
- If an `INVALID` error is reported (e.g., disconnected Relationships, missing mandatory properties, undefined variables), automatically read the error, initiate debugging, and auto-fix the issue.
- Deliver a concise Markdown acceptance report to the user, listing the changes made, Variable configurations applied, and a summary of any errors that were completely resolved.
