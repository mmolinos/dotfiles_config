# Field Discovery And Write Workflow

Use this process to keep the interaction aligned with the current Jira configuration.

## 1. Resolve Site, Project, And Type

1. Resolve the Jira cloud with `atlassian_getAccessibleAtlassianResources`.
2. Default to project `CW` only when the user did not provide a project.
3. Call `atlassian_getJiraProjectIssueTypesMetadata`.
4. Exclude subtask types unless the user explicitly requests a subtask.
5. Put `Task`, `Story`, `Defect`, `Incident`, and `Epic` first when available, followed by other non-subtask types returned by Jira.
6. Validate a custom type answer against the returned metadata.
7. Use the issue type name selected from Jira in `atlassian_createJiraIssue`; do not translate it.

## 2. Build The Available Field Set

Call `atlassian_getJiraIssueTypeMetaWithFields` with `requiredFieldsOnly: false` for the selected project and issue type.

Build two sets:

- `createFields`: every field returned by create metadata.
- `editFields`: fields consistently exposed in `editmeta` by initial-status issues of the exact same type.

To discover `editFields`:

1. Run JQL equivalent to `project = <PROJECT> AND issuetype = "<TYPE>" AND statusCategory = "To Do" ORDER BY created DESC` with a small result limit.
2. Fetch candidates with `atlassian_getJiraIssue`, requesting `expand: names,schema,editmeta,changelog` and the common candidate field IDs.
3. Keep only candidates whose changelog contains no `status` field change. Their current status is therefore the creation status for that workflow.
4. Accept a post-create field only when its edit operation contains `set` in every retained candidate.
5. Prefer at least two retained samples. If only one is accessible, mark the field as conditionally available in the preview and revalidate it immediately after creation.
6. If no untransitioned same-type issue exists, do not ask for post-create-only fields.

Always ask fields marked `required`. For optional fields, limit the normal guided flow to the common fields below. If the user explicitly asks for another field, include it only after validating its metadata.

## 3. Common CW Field Aliases

These aliases have been observed in project `CW`. Treat them only as lookup hints. Resolve fields from current `names` and `schema` metadata on every run, and ignore an alias when the metadata differs.

| Jira field | Field key | Expected value |
| --- | --- | --- |
| Team | `customfield_10001` | Atlassian team ID string |
| Sprint | `customfield_10010` | Numeric sprint ID |
| Story Points | `customfield_10024` | Number |
| Estimation - Backend | `customfield_10045` | Number |
| Estimation - Frontend | `customfield_10046` | Number |
| Parent | `parent` | `{ "key": "CW-123" }` or the create tool's `parent` parameter |
| Reporter | `reporter` | `{ "accountId": "..." }` |

Do not infer field support from these aliases. Field context can differ by issue type.

## 4. Selector Behavior

Use the harness-native structured input tool for selectors when one is available. Otherwise, present the same selector as a concise numbered plain-text question and wait for the user's answer. Keep each label short, put the recommended value first, and explain why it is recommended.

### Sprint

Discover sprint candidates with either approach:

1. Prefer `cw-ai-gateway_jira_inspect` for a read-only list of current board sprints when the board is not known.
2. Fall back to Atlassian JQL: query recent project issues with `Sprint IS NOT EMPTY`, request the Sprint field, deduplicate sprint objects by ID, discard every closed sprint, and sort active first followed by future sprints by ascending start date.

Show at most three active or future sprint choices, then `Omitir`, and allow the user to enter another sprint name. Validate custom names with JQL and read a matching issue's Sprint field to resolve both its numeric ID and current state. Reject closed custom sprints before previewing the ticket. Never submit a sprint name as the field value. If a custom sprint is valid but its ID or state cannot be resolved because it has no accessible issues, explain the limitation and ask the user to choose another sprint or omit it.

### Team

Query recent project issues with `Team IS NOT EMPTY`, deduplicate team objects, and show up to five relevant teams. Include `Omitir` when Team is optional. Resolve custom answers by querying exact team name and reading the Team field object. Use the selected team later to rank parent/epic candidates.

### Reporter

Resolve the authenticated Atlassian user with `atlassian_atlassianUserInfo` and offer that user first. Allow a custom name or email and resolve it with `atlassian_lookupJiraAccountId`. If multiple users match, ask the user to choose. Never guess an account ID.

### Parent Or Epic

Query active epics in the selected project, filtered by Team when available, ordered by recent updates. Show up to three choices, then `Omitir`; allow a custom issue key or URL. Fetch the selected issue and verify that its hierarchy can parent the selected issue type.

### Enumerated Required Fields

Convert metadata `allowedValues` into selectors. Use `multiple: true` when the field schema is an array or multiselect. Store each selection in the shape expected by the schema, normally `{ "id": "<option-id>" }`. For a required field with no allowed values returned, stop and explain that Jira metadata did not provide a safe value to submit.

## 5. Estimation Selection

Evaluate estimations in this order:

1. If Backend and Frontend estimation are both in `createFields` or settable `editFields`, ask both numeric values separately. Include `Omitir` for each optional estimate.
2. Otherwise, if Story Points is in `createFields` or settable `editFields`, ask one numeric Story Points value.
3. Otherwise, skip estimation entirely.

Observed CW behavior, for discovery hints only:

- `Task` supports Story Points at creation and split Backend/Frontend estimation during editing. Prefer the split estimates.
- `Story` can expose split Backend/Frontend estimation during editing. Prefer the split estimates when confirmed.
- `Defect` exposes Story Points but not split Backend/Frontend estimation. Ask only Story Points.
- Preserve `0` in all numeric fields.

Do not use these observations to decide field availability. Apply the estimation order above using the current create and edit metadata.

## 6. Collect Work Information

Before drafting, inspect create metadata for `description` and fields named `Initial description...`, then inspect a recent same-type ticket's description. If its stable heading structure differs from the bundled reference, use the current Jira structure.

Choose the content target safely:

1. Use `description` when it exists in `createFields`.
2. Otherwise, use a settable `Initial description...` field only when a same-type example confirms that Jira uses it as the ticket body; format the value according to its schema.
3. Otherwise, explain that the selected type does not expose a safe description field and ask whether to choose another type or proceed without a description.

Ask one broad free-text question first: what needs to change, why, expected result, and relevant links or reproduction details. Generate a proposed summary and map the answer into the selected type's template.

Ask follow-up questions only when mandatory content is still missing. Examples:

- Story or Task: missing reason, scope, or acceptance criteria.
- Defect: missing current behavior, expected behavior, or reproduction steps.
- Incident: missing any template section marked mandatory.

Default ticket text to English because the fallback CW templates are English. Preserve another language when the user explicitly requests it.

## 7. Preview And Confirmation

Show:

- Project and issue type
- Summary
- Full rendered description
- Required fields
- Selected optional fields
- Explicitly omitted fields
- Fields planned for the post-create edit

Use a final selector with `Crear ticket` and `Cancelar`. Do not treat earlier approval of individual values as creation approval.

## 8. Write Sequence

1. Split selected values into `createPayload` and `postCreatePayload` before showing the preview.
2. Put a field in `createPayload` only when it exists in `createFields`. Use the dedicated `parent` argument only when Parent exists in `createFields`.
3. Put a field in `postCreatePayload` only when it passed the initial-status `editFields` check.
4. Call `atlassian_createJiraIssue` with `contentFormat: markdown` and only `createPayload` values in `additional_fields`.
5. Fetch the new issue with `expand: names,schema,editmeta` and request every `postCreatePayload` field.
6. Remove any post-create field whose actual edit operation does not contain `set`, and report it as unavailable on the created issue.
7. Call `atlassian_editJiraIssue` once with the remaining `postCreatePayload`.
8. Fetch the created issue again and verify every requested field.

If creation succeeds but editing fails, keep the created ticket, report its URL and the exact unapplied fields, and ask whether the user wants a retry. Never create a duplicate automatically.

## 9. Projects Other Than CW

Do not reuse CW field aliases or type-specific templates blindly. Discover fields and descriptions from the requested project. If no stable same-type template is observable, use the generic fallback template and identify it as a fallback in the preview.
