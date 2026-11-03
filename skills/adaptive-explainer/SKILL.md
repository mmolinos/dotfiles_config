---
name: adaptive-explainer
description: Explain complex topics, code, and AI outputs with ASD-STE100-inspired clarity and Mermaid-first diagrams. Use when writing PR descriptions, ADRs, technical guides, backend designs, API documentation, runbooks, or explanations of frontend and backend changes, as well as visual explainers, interactive HTML, and narrated video. Apply to technical communication across engineering work, even when no visual artifact is requested. Choose the smallest useful format; complement implementation and testing workflows rather than replacing them.
---

# Adaptive Explainer

Make understanding easier, not merely the output more impressive. Inspired by Andrej Karpathy's progression from clear writing to diagrams, interactive pages, and bespoke explainer videos, treat each format as a tool rather than an automatic upgrade.

Apply this approach across technical documents and engineering work, not just frontend artifacts. Use clear writing for PR descriptions, ADRs, guides, and change summaries. Add Mermaid diagrams when they clarify architecture, data flow, or behavior. Do not turn an engineering task into an unrelated frontend project.

## 1. Establish the learning goal

Identify:
- The topic and source material.
- The audience and their existing knowledge.
- What the reader should understand or be able to do afterward.
- Any requested format, depth, time limit, or delivery path.

Use context already provided. If the topic or a consequential constraint is missing, ask a short question. Otherwise, state a reasonable assumption and proceed. Do not turn a simple explanation into a requirements interview.

Read relevant source files or documents before explaining them. For AI-generated material, verify important claims rather than treating fluent text as evidence. Separate source facts, assumptions, and unresolved questions. Cite sources when available; never invent citations. Preserve conditions, units, exceptions, and uncertainty when simplifying.

## 2. Choose the smallest useful format

Honor an explicit format request. Otherwise, use this guide:

| Learning need | Default format | Why it helps |
| --- | --- | --- |
| A definition, short answer, or procedure | Clear text | Fast to read and easy to reuse |
| A PR description | Concise Markdown using the repository's PR template | Helps reviewers see the change, reason, and verification |
| An ADR, technical guide, runbook, or backend change explanation | Markdown using the existing document template | Fits repository review and maintenance |
| Relationships, architecture, flow, or sequence | Text plus a Mermaid diagram | Makes structure visible |
| Exploring parameters, comparing states, or stepping through a process | Self-contained HTML | Lets the learner test a mental model |
| Understanding motion, change over time, or a narrated derivation | Video, when requested and feasible | Synchronizes visuals and explanation |

Start with clear text when no richer format has a clear benefit. Do not create every format, a web app, or a video just because tools are available. Explain a non-obvious format choice in one sentence. When an artifact is requested, produce the artifact rather than stopping at a plan, unless a real capability or permission blocks it.

## 3. Build a correct mental model

Organize the explanation around the learning goal:
1. Give the main idea in one or two sentences.
2. Define only the terms the audience needs.
3. Explain the mechanism in small steps.
4. Show one concrete example or worked case.
5. Identify a limitation or common misunderstanding when relevant.
6. End with a short takeaway or a useful comprehension question.

Adapt this structure for short answers and procedures; do not force six headings into every response. Label analogies as analogies and explain where they stop matching reality. Introduce one new idea at a time. Use the same names in the prose, diagrams, controls, and narration.

## 4. Write with STE-inspired clarity

Default to a relaxed, roughly "80% of the way to ASD-STE100" style. This describes a writing preference, not a measured compliance score. Keep natural explanations and necessary technical vocabulary.

Use these rules from the supplied Simplified Technical English overview:
- Use short sentences, familiar words, and explicit subjects.
- Use one term for one thing. Do not switch synonyms for variety.
- Prefer active voice. Use command verbs in procedures: "Close the valve."
- Put one instruction in each procedural sentence, except for necessary simultaneous actions.
- Aim for at most 20 words in a procedural sentence and 25 words in a descriptive sentence.
- Keep descriptive paragraphs to one topic and at most six sentences.
- Keep noun clusters to at most three words where possible. Define longer technical names instead of making them ambiguous.
- Prefer simple verb forms. Avoid progressive and perfect forms when a simpler form preserves the meaning.
- Make pronoun references clear. Repeat the noun when "it", "this", or "they" could refer to different things.
- Use vertical lists for complex sequences or conditions.

Prefer plain alternatives when the meaning is unchanged:

| Avoid | Prefer |
| --- | --- |
| commence | start |
| utilize | use |
| prior to | before |
| approximately | about |
| in order to | to |
| ensure | make sure |
| replenish | fill, when that accurately describes the action |

Example:
> Original: It is imperative that the operator ensures the hydraulic reservoir is replenished prior to commencing operation.
>
> Clearer: Make sure that the hydraulic reservoir is full before you start the operation.

Do not apply replacements blindly: "refill to the specified level" may be more accurate than "fill". Keep exact identifiers, code, quotations, and necessary technical names intact. Do not put approved words in uppercase in normal prose; the image uses uppercase as a dictionary convention.

### Strict requests and safety limits

For an explicit strict ASD-STE100 request, ask for the required edition and approved dictionary or technical-name list if unavailable. Treat the image as an overview, not the complete specification. Without those resources, offer an STE-inspired rewrite and clearly say that formal compliance has not been verified. Do not invent dictionary approvals or claim certification.

For safety instructions, put the instruction before the explanation of the risk. Preserve existing warnings and cautions. In the overview, WARNING concerns injury and CAUTION concerns damage. Do not invent a hazard classification or turn an unverified explanation into operational safety guidance. High-stakes procedures require authoritative source material and qualified review.

## 5. Produce the selected artifact

### Text

Deliver the explanation directly. Use numbered steps for procedures and a small glossary only when necessary. Preserve distinctions that matter more than a word-count target.

### PR descriptions

Inspect the actual diff, relevant tests, and repository PR template before drafting. Summarize what changed and why, then state verification that was actually performed. Call out material behavior changes, risks, rollout or migration notes, and follow-up work when relevant.

Use the repository's headings and conventions when available. Otherwise, keep the description scannable with a short summary, rationale or key details, and tests/verification. Include only sections that add value. Link related issues when supplied. Do not invent motivation, test results, user impact, approvals, or claims of completeness. Keep the description focused on reviewer understanding; do not restate every changed file. Add a Mermaid diagram only when it materially clarifies a non-trivial flow or architecture change, not as a default decoration.

### ADRs

Follow the repository's ADR template, numbering, location, and status conventions. If no template exists, use: title and status, context, decision drivers, considered options, decision, consequences, and verification or rollout considerations when relevant.

Explain why the decision fits the constraints. Include rejected alternatives and their tradeoffs. Distinguish proposed decisions from accepted ones; do not invent approval. Use a Mermaid diagram when the decision changes boundaries, dependencies, or runtime behavior. Keep the reasoning understandable without the diagram.

### Technical guides

Follow existing documentation conventions. State the audience, goal, prerequisites, and scope. Separate conceptual explanations from how-to steps and reference details. Include a concrete example, expected outcomes, and relevant troubleshooting or limitations. Verify commands and configuration against the source; label untested examples.

Use Mermaid for architecture and request-flow diagrams when they aid understanding. For runbooks, preserve operational safeguards, failure conditions, and recovery instructions from authoritative sources. Do not invent runnable commands or recovery steps to fill gaps.

### Backend work

For backend design or implementation tasks, apply this skill to design notes, documentation, and the explanation of changes. Continue to follow the project's implementation, architecture, and testing skills; this skill does not replace them or authorize extra code changes.

Explain relevant service boundaries, API contracts, data ownership, transactions, asynchronous flows, and failure behavior. Include only concerns that matter to the task. Preserve distinctions such as retries versus idempotency and authentication versus authorization. Link statements about existing behavior to inspected code or tests, and distinguish proposed behavior from implemented behavior.

Use a Mermaid sequence diagram for interactions between services, a flowchart for control or data flow, an ER diagram for data relationships, or a state diagram for lifecycle transitions. Show failure paths when they are important to the explanation. Do not add unrelated diagrams or documentation files to a code-only task; keep the change summary proportionate.

### Diagrams and images

Use Mermaid by default for diagrams in ADRs, technical guides, backend explanations, and other supported scenarios. Put editable Mermaid source in fenced `mermaid` blocks in Markdown. Choose a diagram type that matches the concept and use syntax supported by the target renderer.

Use ASCII only when the delivery environment requires plain text or the user requests it. Use SVG or another image format when Mermaid cannot express the required illustration, when animation requires it, or when explicitly requested. Lack of a renderer alone is not a reason to replace Mermaid source. Explain a non-obvious exception briefly. For HTML or video that needs rendered diagrams, retain the Mermaid source where practical and use an available local renderer rather than adding a remote dependency by default.

Label nodes, arrows, axes, units, and state changes where relevant. Keep labels concise and include a brief text equivalent for accessibility.

Verify the diagram against the source. Distinguish known relationships from inferred ones. Avoid decorative complexity. Render or syntax-check when tools permit; otherwise identify that the diagram has not been rendered. Never report an image file as generated if only diagram source exists.

### Interactive HTML

Prefer one self-contained HTML file with inline CSS, JavaScript, and SVG. Avoid frameworks, build steps, remote fonts, and external assets unless the task needs them. Use one meaningful interaction, such as a step control, slider, or state comparison, that demonstrates the concept.

Include:
- A visible learning goal, concise explanation, and clear labels.
- Semantic headings, keyboard-operable controls, readable contrast, and responsive layout.
- A text equivalent for important visuals.
- Reduced-motion support for animation and pause controls for ongoing motion.
- An explanation of assumptions and whether a simulation is illustrative or quantitatively accurate.

Keep the core explanation readable without JavaScript. Insert untrusted source text as text, not executable markup. Do not embed API keys, trackers, or private source material without permission. Keep private content local; do not publish an artifact by default.

Test available controls, boundary values, and reset behavior. Check the browser console and a narrow viewport when browser tools are available. Otherwise, perform a source review and state that interactive behavior was not browser-tested.

### Explainer video

First check available rendering and audio tools. Use geometric, step-by-step visual reasoning when a user requests a "3b1b-style" explainer; do not impersonate the creator, clone their voice, or copy their branding.

Create a short storyboard with scene purpose, visual action, narration, and timing. Match the requested duration. Explain each visual change as it occurs. Define mathematical symbols before use. Keep music and transitions subordinate to comprehension.

Choose the simplest available production path, such as Manim or rendered SVG scenes plus FFmpeg. Do not install dependencies or start paid jobs without appropriate permission. Inspect output paths before writing and do not overwrite user files without approval.

For narration:
- Use an existing, explicitly authorized provider such as ElevenLabs only with the user's consent to transmit the script.
- Read credentials through environment variables or the approved secret mechanism. Never request secrets in chat or embed them in generated files, browser code, or logs.
- If paid narration is unavailable, offer an installed local TTS engine, such as Piper, or a silent version with captions. Check availability instead of assuming local tools exist.
- Use a generic voice; do not clone an identifiable person's voice without authorization.

Render the video if tools and permissions allow. Include captions or a transcript, and check duration, synchronization, legibility, audio, and playback. If rendering is blocked, deliver the storyboard, narration, and feasible source files. Clearly label them as unrendered assets, not a finished video.

## 6. Review and deliver

Before delivery, check:
- Does the output answer the actual question for this audience?
- Are facts, units, conditions, and limitations preserved?
- Do visuals and interactions teach the same model as the text?
- Is the language concise without changing the meaning?
- Are artifacts accessible and free of exposed secrets?
- What was actually rendered or tested, and what remains unverified?

Return the explanation or artifact path, a short statement of what it teaches, and any relevant verification limits. Provide opening or rendering instructions when needed. Do not bury the result under a long process report or claim formal STE compliance from a style-only review.
