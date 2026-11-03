# CW Description Templates

These fallback structures come from observed project `CW` tickets and Jira's initial-description fields. Treat them as examples, not current Jira configuration. Prefer a template discovered at runtime from current metadata and recent same-type tickets. Use the type-specific templates below only for project `CW`; for other projects use their observed template or the generic fallback. Fill templates from the user's explanation and never invent facts.

Keep the Jira headings in English unless the user explicitly requests another language. Do not leave instructional placeholders such as `<mandatory>` in the final description. Ask for missing mandatory information before previewing the ticket.

## Story And Task

Jira uses the same initial-description structure for Story and Task through `Initial description for stories and tasks`. Verified examples include `CW-66017` and `CW-68104`.

Use the complete structure when the user provides enough detail. Keep optional sections concise and use `N/A` only when the user confirms they do not apply.

```markdown
## Why

<Business or technical reason>

## What

<Scope and requested outcome>

## Affected functionality

<Affected flows, users, services, or N/A>

## Acceptance criteria

- [ ] <Observable criterion>

## Tracking criteria

<Analytics or tracking requirements, or N/A>

## Development Breakdown (Optional)

### Implicated Repo(s)

<Repositories, if known>

### Outside-code development needed

<Configuration or external changes, or N/A>

### Basic development instructions

<Known implementation guidance without inventing a solution>

### Documentation

<Relevant documentation or N/A>

### Suggested reviewer/adviser

<Person or team, if known>

## Is this change platform-wide?

<Yes or No, with scope>

### If so, was there Ops sign-off for such a change?

<Sign-off information or N/A>

### New Documentation

<Documentation to create or N/A>

### Unit testing

<Expected automated test coverage>
```

For a small Task, retain at least `Why`, `What`, and `Acceptance criteria`. Do not inflate unknown optional sections; use the full template when the work requires product or QA validation.

## Defect

Formal CW defects use the same reproducibility structure as bugs and incidents. Verified example: `CW-67519`.

```markdown
## What

<Concise problem statement and affected context>

## Steps to reproduce

1. <First reproducible step>
2. <Next step>

## Current behavior

<Observed result, including errors when known>

## Expected behavior

<Correct expected result>

## Known workaround

<Workaround or N/A>
```

Require `What`, `Steps to reproduce`, `Current behavior`, and `Expected behavior`. Allow `Known workaround` to be `N/A`.

## Incident

Manual incidents use the same core structure as Defect. Verified example: `CW-68999`. Automated incident-reporter metadata is integration-generated and must not be fabricated.

```markdown
## What

<Production impact and affected users or systems>

## Steps to reproduce

1. <First reproducible step>
2. <Next step>

## Current behavior

<Observed production behavior>

## Expected behavior

<Expected production behavior>

## Known workaround

<Workaround or N/A>
```

Require all sections except `Known workaround`. Also collect every Jira field marked required by Incident metadata, such as impact, urgency, detected-by, severity, region, or impacted programs.

## Epic

Jira's `Initial description for epics` uses this structure. Verified example: `CW-67044`.

```markdown
## Why

<Strategic or business reason>

## What

<Outcome and scope covered by the epic>
```

## Other Types

When no verified template exists, use a conservative structure and label it in the preview as a fallback template:

```markdown
## Context

<Why this work is needed>

## Scope

<What must be delivered>

## Acceptance criteria

- [ ] <Observable criterion>
```

Before using the fallback, inspect recent same-type tickets with non-empty descriptions. If a stable current structure is evident, follow it and mention the example ticket keys in the preview.
