---
name: sre-postmortem
description: Generate a blameless SRE postmortem from a Slack incident channel, publish to Confluence, and create Linear action items. Use when an incident has resolved and the user wants a structured postmortem — they will provide the incident channel name or key details.
---

## Prerequisites

Halt and ask the user if any are missing:
- `SLACK_BOT_TOKEN` — scopes: `channels:history`, `channels:read`
- `CONFLUENCE_DOMAIN` — e.g. `yourco.atlassian.net`
- `CONFLUENCE_API_TOKEN` — write access to the `SRE` space
- `LINEAR_API_KEY` — write access to project INC

## Gotchas

- **Confluence uses storage format (XHTML), not Markdown.** Convert the drafted postmortem before POSTing. The REST endpoint is `POST /wiki/rest/api/content`.
- **Slack `conversations.history` returns max 1000 messages per page.** Use `response_metadata.next_cursor` to paginate. Messages with `reply_count > 0` need separate `conversations.replies` calls for their threads.
- **Linear issues require a `teamId`, not a project key.** Fetch it first: `query { teams { id name } }`, then match on the INC team.
- **Never invent missing data.** Mark unknowns as `[TBD — not captured during incident]`.
- **No real names, no PII.** Every person is referenced by role (primary, secondary, IC). Strip social chat, jokes, health, family, and compensation talk from the transcript before drafting.

## Rules (apply throughout)

- **Blameless.** Attribute failures to systems and actions, not people. "The deploy introduced a misconfigured flag" not "Alice's bad deploy."
- **5 Whys.** Start at the root cause. Each answer becomes the next "why." The fifth answer must surface a systemic fix. Never stop at "human error" — ask why the error was possible.
- **Action items concrete.** Every item has a clear "done" state. "Add alert on auth-latency p99 > 500ms" not "investigate auth performance."
- **Runbook links.** Link `SRE/Runbooks/{service}` in Confluence for each affected service. Flag services without a runbook as a finding.
- **Gaps marked.** Unknown fields get `[TBD — not captured during incident]` — never guessed.

## Workflow

### Stage 1: Gather metadata

**Input:** User provides a Slack channel name (`#inc-YYYYMMDD-shortname`) or enough detail to derive it.

**Action:** Collect the following. If any field is unknown, proceed with the best available info:
- Incident start date/time
- Severity (SEV0–SEV3). Default: "Unspecified"
- Affected services
- Primary, secondary, and IC roles

**Output:** Metadata block carried forward to Stage 4.

### Stage 2: Pull the Slack transcript

**Input:** Channel name from Stage 1.

**Action:**
1. Resolve the channel name to a channel ID (use `conversations.list` or parse from the `#inc-YYYYMMDD-shortname` format).
2. Fetch all messages with `conversations.history` — iterate `cursor` from `response_metadata.next_cursor` until exhausted.
3. For every message where `reply_count > 0`, fetch the thread with `conversations.replies`.
4. Sort all messages by timestamp.

**Output:** Full transcript as a local file. Each entry: `{ts, user, text, thread_ts}`.

### Stage 3: Extract the incident narrative

**Input:** Transcript from Stage 2.

**Action:** Read the transcript and extract only incident-relevant content. Skip social chat, stand-ups, jokes, personal content.

Extract these into a structured narrative:
- **Timeline (PT):** Every significant event — detection, diagnosis, mitigation, resolution. Each entry: timestamp, description, acting role. Mark estimated timestamps with `(approximate)`. Look for alert names, metric thresholds, error messages, runbook references, dashboard URLs, PR links, and deploy references — include these in timeline entries.
- **Customer impact:** Services affected, degradation type (down/degraded/slow), duration, dollar impact if stated.
- **Root cause:** The specific technical failure — commit, config change, dependency failure, capacity breach.
- **Contributing factors:** Conditions that let the root cause become an outage (missing alerts, deploy-on-Friday, stale runbook, etc.).

**Output:** Narrative draft used as input to Stage 4.

### Stage 4: Draft the postmortem

**Input:** Narrative from Stage 3, metadata from Stage 1.

**Action:** At the start of this stage, load [postmortem-template.md](references/postmortem-template.md) for the section-by-section structure. Fill every section from the narrative, applying all Rules above.

Verify before proceeding:
- Every section is filled or marked `[TBD — not captured during incident]`
- Zero real names appear
- Zero personal/sensitive content leaked
- Every affected service has a runbook link or a flagged gap

**Output:** Complete postmortem in Markdown.

### Stage 5: Publish to Confluence

**Input:** Postmortem from Stage 4, metadata from Stage 1.

**Action:**
1. Convert the Markdown postmortem to Confluence storage format (XHTML).
2. Check for an existing page: `GET /wiki/rest/api/content?title={YYYYMMDD}+—+{shortname}&spaceKey=SRE`. If a page exists, ask the user whether to update or abort.
3. POST to `/wiki/rest/api/content` with `spaceKey: "SRE"`, title `{YYYYMMDD} — {shortname}`, body in storage format, under ancestor `SRE/Postmortems`.

**Output:** Published Confluence page URL.

### Stage 6: Create Linear action items

**Input:** Action items table from Stage 4, Confluence URL from Stage 5.

**Action:**
1. Fetch the team ID via Linear GraphQL: `query { teams { id name } }`. Match on the INC team.
2. For each action item, create an issue with `mutation IssueCreate`:
   - `teamId`: fetched team ID
   - `title`: action item text
   - `description`: link to the Confluence postmortem
   - `priority`: SEV0/SEV1 → 1 (Urgent), SEV2 → 2 (High), SEV3+ → 3 (Medium)

**Output:** List of Linear issue URLs.

### Stage 7: Final verification

Run through the checklist:
- [ ] Every postmortem section is filled or marked `[TBD — not captured during incident]`
- [ ] Zero real names appear (roles only)
- [ ] Zero personal/sensitive chat content leaked
- [ ] Every affected service has a runbook link or a flagged gap
- [ ] Confluence page is accessible
- [ ] Every action item has a corresponding Linear issue linking back to the postmortem

Report: Confluence URL, Linear issue URLs, and any sections that remain TBD.
