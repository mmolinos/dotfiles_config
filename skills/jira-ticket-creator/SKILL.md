---
name: jira-ticket-creator
description: Interactively create Jira tickets with Atlassian MCP using type-aware fields, selectable recent sprints, teams, reporters, estimations, parent epics, and project description templates. Use when users ask to create a Jira ticket, task, story, defect, incident, epic, or another standard issue type and want a guided preview-and-confirm workflow.
---

# Jira Ticket Creator

Create Jira tickets through a guided, type-aware workflow. Discover Jira configuration at runtime, ask only relevant questions, render the appropriate description template, and require confirmation before writing.

## Requirements

Requires Atlassian MCP write access and a harness-supported way to ask the user for input. `cw-ai-gateway` is optional and read-only.

## Hard Constraints

- Use Atlassian MCP as the source of truth and for every Jira write.
- Use `cw-ai-gateway_jira_inspect` only for optional read-only discovery or summarization.
- Never create or modify a ticket before showing a complete preview and receiving explicit confirmation.
- Never ask for or send a field that Jira does not expose for the selected issue type.
- Use the harness-native structured input tool for finite choices when one is available. Keep custom answers enabled and include `Omitir` for every optional field. If no structured input tool is available, ask the same choices as one concise plain-text question and wait for the answer.
- Revalidate names and IDs at runtime. Treat IDs documented in references as field aliases, not immutable configuration.
- Never offer or submit a closed sprint. Sprint selectors may contain only active or future sprints whose state was validated at runtime.
- Do not offer or create the Jira issue type named `Platform Engineering Issue`. Use `platform-request-creator` when the user asks for a Platform Request.

## Core Workflow

1. Resolve the Jira site and project. Default to project `CW` unless the user specifies another project.
2. Fetch project issue types with `atlassian_getJiraProjectIssueTypesMetadata`. Exclude subtasks and `Platform Engineering Issue`. Present common types as a single-select list, with a custom answer for any other available type except the excluded ones.
3. Fetch all create fields for the selected type with `atlassian_getJiraIssueTypeMetaWithFields` and inspect multiple initial-status issues of the same type for post-create edit fields. Follow [field-discovery.md](references/field-discovery.md).
4. Ask every required field and ask supported common fields in this order: sprint, team, reporter, estimation, parent/epic, then priority and any remaining required type-specific fields. Offer `Omitir` only when Jira marks a field optional or supplies a valid default.
5. Present up to three relevant assignable sprints in a selector, ordered active first and future second. Never include closed sprints. Include `Omitir` and allow a custom sprint name. Resolve the selected name to its Jira sprint ID and revalidate that it is active or future before preview and creation.
6. Resolve team, reporter, and parent values to their Jira IDs. Never submit display names where Jira expects an ID.
7. Ask the user to describe the requested work. Draft a concise summary and a description using the current Jira template when one is discoverable, otherwise use [description-templates.md](references/description-templates.md). Ask focused follow-up questions only for missing mandatory template information.
8. Show a final preview containing the issue type, project, summary, rendered description, all selected fields, omitted fields, and any values that require a post-create update.
9. Ask `Crear ticket` or `Cancelar`. Create only after `Crear ticket` is selected.
10. Call `atlassian_createJiraIssue` with create-compatible fields only. Recheck the created issue's `editmeta`, then call `atlassian_editJiraIssue` for selected fields that are actually editable on that issue.
11. Fetch the created issue with `atlassian_getJiraIssue`, verify the requested values, and return its key and URL. Report any failed post-create field without deleting or recreating the issue.

## Estimation Rules

- If both `Estimation - Backend` and `Estimation - Frontend` are editable for the selected type, ask for both and apply them after creation when necessary.
- Otherwise, if `Story Points` is available, ask for a single `Estimación (Story Points)` value.
- Otherwise, do not display an estimation question.
- Preserve numeric zero. Treat `0` as an explicit estimate, not as an omitted value.

## Required Fields

For type-specific required fields such as Incident impact, urgency, severity, region, or detected-by, render `allowedValues` as selectors. Do not replace them with free text and do not offer `Omitir`.

## References

- [field-discovery.md](references/field-discovery.md) - Runtime field detection, selectors, value resolution, and write sequencing
- [description-templates.md](references/description-templates.md) - Verified CW description structures by issue type
