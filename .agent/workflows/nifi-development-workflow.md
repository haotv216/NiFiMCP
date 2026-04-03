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

## Step 3: Implementation Guide
- Formulate a **detailed, step-by-step Implementation Guide** for the user in Markdown format.
- DO NOT use flow creation tools (e.g., `create_nifi_process_group`, `create_nifi_processors`, `create_nifi_connections`).
- Ensure the guide specifies exactly:
  1. The Processors the user needs to add on the NiFi UI (with their specific class Types).
  2. The exact Properties and Expression Language (EL) variables to configure for each Processor.
  3. The exact Relationships for Connections (Success, Failure, etc.) and what should be Auto-Terminated.
  4. Any required Process Group Variables that must be added.
- Wait for the user to confirm they have manually completed the configuration on the NiFi Canvas.

## Step 4: Validation & Debugging
- After the user confirms they have manually created/updated the flow, use `get_process_group_status` or `analyze_nifi_processor_errors` to check the Health status and queue size of the modified flow.
- Ensure all processors are in a `VALID` state. Specifically, **cross-check Process Group Variables** to confirm no Expression Language references an undefined variable (which would trigger a missing variable warning).
- If an `INVALID` error is reported (e.g., disconnected Relationships, missing mandatory properties, undefined variables), automatically read the error and instruct the user on how to fix it via the UI.
- Deliver a concise Markdown acceptance report to the user, listing a summary of validations and any errors that were completely resolved.
