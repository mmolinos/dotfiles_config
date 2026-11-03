---
name: adversarial-pr-review
description: Run an adversarial multi-model review of a GitHub PR. Fan out several reviewer agents (different models and effort levels, each with its own lens), verify their top findings against the code, merge them into one ranked list, then draft the non-overlapping findings as clear inline comments (or a PR comment when no changed line fits) with the adaptive-explainer writing rules, and post them with gh only after the user confirms the draft. Use when the user asks for an adversarial, multi-model, or "several subagents" review of a PR, or wants findings on wrong claims, missed call sites, contradictions, security, races, retry/replay semantics, or Temporal durability.
---

# Adversarial PR review

Goal: find real defects with evidence, not style nits. Several independent reviewers each attack the PR from a different lens. You verify their claims before passing them on.

## 1. Prepare

1. Read the PR metadata and the file list:
   `gh pr view <N> --json title,body,author,headRefName,headRefOid,additions,deletions,files`
2. Fetch the head without checking it out, so the working tree stays untouched:
   `git fetch origin <headRefName>` and record `SHA=<headRefOid>`.
3. Note whether the PR description is empty. If it is, say that the review judges the code and docs as written.

## 2. Choose reviewers

Use the models and efforts the user names. Map them to providers:

| User says | How to launch |
|---|---|
| Opus / Sonnet / Haiku / Fable | `Agent` tool, `model` + `effort` (the user asked for the effort explicitly) |
| GPT "Luna", "Sol", "Terra", Gemini, others | Paseo `create_agent`, provider `opencode/github-copilot/<model-id>`, `settings: { modeId: "plan", thinkingOptionId: "<effort>" }` |

- Always call `mcp__paseo__list_providers` / `list_models` first. Do not guess model ids (e.g. "Luna 6" = `gpt-6-luna`, "Sol 6.1" = `gpt-6.1-sol`).
- Use OpenCode `plan` mode for Paseo reviewers: they stay read-only.
- If the user names no models, default to 3–4 reviewers on different model families.

Give each reviewer one **primary lens** so their work overlaps less. They still cover every category:

1. Temporal / durability: determinism, workflow ids and reuse policy, activity retry and idempotency, cancellation, continue-as-new, worker restart, workflow registration and bundle.
2. Security and races: authz, injection, PII or secrets in logs and audit stores, concurrent launches, Mongo idempotency and unique keys.
3. Missed call sites and contradictions: every registry, switch or map over the new concept; constants defined twice; openapi vs schema vs UI vs backend.
4. Wrong claims: every factual statement in docs, comments, test names and openapi, checked against code, plus repo guardrails (CLAUDE.md).

## 3. Launch

- Fill [reviewer-prompt.md](reviewer-prompt.md) (PR number, repo path, SHA, lens) and pass the **full text inline** as the prompt. Do not point reviewers at a file in `/tmp`: Paseo agents ask for permission for every external directory.
- Launch all reviewers in **one message** so they run in parallel. Agent-tool reviewers run in the background.
- Tell the user right away: who runs, which lens, and an estimate (roughly 10–20 min for medium, 25–45 min for xhigh on a PR of about 5k lines). Do not poll; wait for the notifications.
- If a Paseo reviewer still asks for permission to read a file, answer `allow_once` (narrowest scope). Never grant write permissions.
- When a Paseo report arrives truncated, get the rest with `mcp__paseo__get_agent_activity` (`limit: 1`).
- Report each reviewer briefly as it arrives. Label its findings as **its claims, not yet verified**.

## 4. Verify and merge

Before the final report, check the top findings yourself against the PR-head code:

- Read with `git show "${SHA}:path"`. In zsh, always use braces and quotes: `$SHA:src` is read as a `:s` modifier and breaks the path.
- Trace each blocker or major end to end (caller → callee → persistence). Record `path:line`.
- Downgrade a finding when it is an existing repo-wide pattern, not something the PR introduced (e.g. no guards on any controller in the module). Mention it as a note.
- When one reviewer marks something "OK" that another flags, trace that path yourself. An "OK" can cover only part of the path, e.g. "not logged" while it is still stored in an audit collection.
- When a reviewer comes back thin, say so. Do not count its silence as a pass.

Final report (write it with the same `adaptive-explainer` rules as the comments in 5a):
- Verdict first, in one or two sentences.
- Findings grouped as Blockers / Major / Minor (wrong claims and contradictions) / Minor (missed call sites) / Guardrails. Give each one `path:line`, the failure scenario, the fix, and which reviewers found it.
- "Checked and fine" list.
- Notes: downgraded items, unverified claims, weak reviewers.
- Say that nothing was posted to GitHub.
- End by **proposing** to publish the findings (section 5). Always offer this, even when the user did not ask for comments.

## 5. Propose comments, wait for confirmation, then post

You are the orchestrator: reviewers never post. You draft every comment, and **nothing goes to GitHub until the user explicitly confirms the draft.** Posting publishes under the user's name and notifies the author.

### 5a. Build the draft

1. **Check existing comments for overlap** (Copilot, other bots, other reviewers, the user's own comments):
   ```
   R=repos/<owner>/<repo>
   gh api $R/pulls/<N>/reviews --paginate
   gh api $R/pulls/<N>/comments --paginate -q '.[] | "\(.id) \(.user.login) \(.path):\(.line // .original_line)\n\(.body)"'
   gh api $R/issues/<N>/comments --paginate
   ```
   Skip findings that are already covered. Keep a finding that is related but has a **different mechanism and fix**; in it, refer to the existing comment.
2. **Choose where each finding goes.**
   - **Inline comment**: the defect or the wrong claim is on a line the PR changed. Every line of a new file counts. For a modified file, check the hunk with `git diff -U0 main...$SHA -- <file>`. Put the comment on the line where a reader needs it (the faulty call, the wrong doc sentence), not just any changed line in the file.
   - **PR comment**: no changed line is a useful anchor. Typical cases: the defect is in a file the PR did not touch, it spans many files, or it is a design or repo-wide note. Collect these in one PR-level comment. Do not force them onto an unrelated changed line.
3. **Write the bodies with the `adaptive-explainer` skill.** Load it (Skill tool, or read `~/.agents/skills/adaptive-explainer/SKILL.md`) and apply its writing rules. The reader is the PR author: they know the code, but they did not see the review.
   - Use this shape for every comment (inline or PR):
     1. `**[severity] One-sentence claim.**`: state the defect, not the topic.
     2. **Why:** explain the mechanism in small steps, with `path:line` for each step. Use a vertical list when it has more than two steps.
     3. **Example:** one concrete failure scenario with real values (batch size, attempt numbers, ids).
     4. **Fix:** one line. Add a second option only when there is a real tradeoff.
   - Write in the STE-inspired style: short sentences, active voice, explicit subjects, one term for one thing. Keep identifiers, code and quotes exact.
   - Keep conditions and uncertainty. Mark `PLAUSIBLE` findings as such ("can happen when…"), and do not state them as fact.
   - Pick the smallest format. Most comments are text only. Add a Mermaid diagram (GitHub renders `mermaid` blocks in comments) only when the failure is a timing or sequence problem across components, e.g. two activity attempts racing or a retry that re-reads changed data. Prefer `sequenceDiagram`. Keep it to about 8 messages, include the failure path, and verify it against the code.
   - Do not repeat the same evidence in several comments. Link to the other comment, or put the shared part in the PR comment.
   - Check every `path:line` you cite.
4. **Save the draft** with a script outside the repo (e.g. `/tmp/<pr>-review/build.py`, run with `python3 -I`):
   - `review.json`: `{ "commit_id": SHA, "event": ..., "body": summary, "comments": [{ "path", "line", "side": "RIGHT", "body" }] }`
   - `pr-comment.md`: the PR-level comment, if there is one.

### 5b. Show the draft and stop

Show the user:
- A table: finding, severity, destination (`inline path:line` or `PR comment`), and why that destination.
- What was skipped as overlap, and with which existing comment.
- The proposed review type: `COMMENT`, `REQUEST_CHANGES`, or `APPROVE`. Default to `COMMENT`. Check `gh api user -q .login` against the PR author: authors cannot request changes on their own PR.
- The scope, e.g. majors only, or everything.
- The draft file paths, so the user can read the full bodies.
- One full example body (the most important finding), so the user can judge the tone and format before reading the rest.

Then **end the turn and wait.** Do not post on silence, on an implied "looks fine", or on a confirmation from a subagent. Only the user's explicit confirmation of this draft counts. If the user changes the scope, destinations or wording, rebuild the draft and show it again. A confirmation covers only the draft it was given for.

### 5c. Post (after confirmation only)

1. Inline comments, as one review (one notification):
   `gh api -X POST $R/pulls/<N>/reviews --input review.json`
2. PR comment, if any:
   `gh pr comment <N> --body-file pr-comment.md`
   If the user prefers a single notification, put it in the review `body`.
3. Verify. The reviews endpoint returns `line: null`, so check the anchors with:
   `gh api $R/pulls/<N>/comments --paginate -q '.[] | select(.pull_request_review_id==<id>) | "\(.path):\(.line // .original_line)"'`
4. Report the review URL, the PR comment URL, what was posted and where, what was skipped as overlap, and any judgment calls.
