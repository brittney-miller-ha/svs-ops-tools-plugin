---
name: file-svs-ticket
description: >
  Files a new SVS Jira ticket (Initiative, Epic, Task, or Sub-task) on behalf of SVS team
  members, following SVS Jira best practices and filing guidelines. Use this skill any time
  someone asks to "create a Jira ticket", "file a ticket", "open an epic", "add a task",
  "log a project", "track this work", or any similar request to create a new item in the SVS
  Jira project. Also trigger when the user describes a project, problem, or piece of work and
  says something like "can you track that?", "let's make a ticket for this", or "I need to
  file something for this." Claude gathers context, infers priority and related work from the
  conversation, generates a natural-sounding description using the SVS template, and returns
  the live Jira link.
---

# File an SVS Jira Ticket

This skill creates well-formed SVS Jira tickets that follow the team's filing guidelines,
using the SVS description template and priority framework. The goal is a ticket that gives
anyone reading it a clear picture of the work — without sounding like it was filled out by a
robot.

**References:**
- [[WIP] SVS - Jira Filing Guidelines](https://docs.google.com/document/d/1rmXc0pGEkHAD3RsQmOgnWXHdqnJULA74EtXDCha5CX8/edit)
- [SVS Jira Best Practices](https://instacart.atlassian.net/wiki/spaces/cx/pages/5486739635/SVS+-+Jira+Best+Practices)

---

## Step 1: Atlassian cloud ID

**Cloud ID:** `af51c917-4866-4b81-91f2-4aaa31b02f3e`

Use the `getAccessibleAtlassianResources` MCP tool to confirm this if needed. Cache it for the session.

---

## Step 2: Gather required fields

All four of these fields are **required** before filing. Pull answers from the conversation
first, then ask for anything missing — in one friendly message, not a form.

### Required fields

**1. Work type**
Options (SVS hierarchy: Initiative → Epic → Task):
- **Initiative** — Active SVS program or major project area for H1/H2 (e.g. Receipt Auditing, Caper, Catalog Ops)
- **Epic** — A specific project, attached to an Initiative
- **Task** — Component work within a project, attached to an Epic

If the user's language is ambiguous, suggest the most likely type and confirm.

**2. Components** *(select all that apply)*
Present these as options and ask the user to pick any that apply. Multiple selections are fine.

> AI · Automation · BPO · Caper · Communication · Metrics · Quality Assurance ·
> Receipt Auditing · Tooling · WFM · Workflow Development

**Important:** Only the components listed above currently exist in the SVS Jira project and
can be set via the API. Do not offer or attempt to set any other component names (e.g.
Documentation, Invest in People, HITL, Catalog Ops, Emerging Programs, Promote Cultural
Norms, Research, ML/AI Data Validation, Instacart Task Platform) — the API will reject them
with a permissions error. If the user's work clearly calls for a component not in the list,
note it in the ticket description instead and let the user know they can ask a Jira project
admin to create it.

When you have enough context, pre-select the obvious ones and ask the user to confirm or add more:
> "Based on what you've described, I'd tag this with **BPO** and **Workflow Development** — anything else to add?"

**Note:** `Claude-Generated Ticket` is automatically added to every ticket filed through this
skill — no need to ask the user about it. Always use this exact casing (`Claude-Generated
Ticket`) — it must match the component name as it exists in Jira.

**3. Priority** *(P0–P4 — always suggest, never just ask)*
Don't ask "what's the priority?" — read the context and propose one with a brief reason.
See the priority guide in Step 3 below.

**4. Due date**
Always ask if not provided. Use the user's language (e.g. "end of May" → `2026-05-30`).
Due dates should fall on a **weekday (Monday–Friday)**. If the date the user gives lands on
a Saturday or Sunday, round back to the preceding Friday and mention it:
> "May 31 is a Sunday — I've set the due date to Friday, May 29 instead. Sound good?"
For Initiatives and Epics, also ask for a **start date** if not mentioned.

### Optional but helpful fields

| Field | Notes |
|---|---|
| **Title / Summary** | Must have — describe like an email subject line |
| **Assignee** | Default: unassigned |
| **Parent ticket** | Recommend one based on Jira context (see Step 4) |
| **Linked work items** | Ask if there are related tickets to link |
| **Resources** | Links to plans, dashboards, PRDs, Slack threads |

**How to ask:** Group into one conversational message. Lead with what you already know:
> "Got it — a few things before I file this. I'd suggest **P1** given the BPO launch
> timeline. Does that feel right? Also: what's the due date, and should I tag this as
> **BPO + Workflow Development**, or are there other components?"

---

## Step 3: Suggest a priority (always lead with a recommendation)

Read the conversation — and any referenced Slack messages, emails, meeting notes, or Jira
tickets — and propose a specific priority with a short reason. Never silently default.

| Priority | When to use |
|---|---|
| **P0 – Critical** | Large-scale, strategic; aligned to H1/H2 goals; multiple stakeholders; critical to team success |
| **P1 – Important** | Significant projects contributing to team/dept goals; may involve cross-functional work |
| **P2 – Medium** | Moderate scope and value; can deprioritize if P0/P1 needs bandwidth |
| **P3 – Low** | Non-urgent, smaller-impact, stretch goals; deferrable without negative consequences |
| **P4 – De-prioritized** | Parking lot; explicitly deferred to a later date |

**Signals that push priority up:** hard deadline, BPO/vendor launch date, exec visibility,
large user or revenue impact, blocking language ("this is holding us up"), H1/H2 alignment,
multiple cross-functional stakeholders.

**Signals that push priority down:** "nice to have", "when we get to it", "stretch goal",
no named deadline, no named stakeholders, exploratory or optional work.

**Present it like this:**
> "Based on the vendor launch in June and the cross-team dependency, I'd suggest **P1**.
> Does that feel right, or should we go higher/lower?"

If context is thin, default to **P2** and say so:
> "I don't have enough context to be confident here — I've defaulted to **P2**, but let
> me know if this is more or less urgent."

---

## Step 4: Recommend a parent ticket

Before asking the user to supply a parent, search Jira first. Use `searchJiraIssuesUsingJql`
to find open Initiatives and Epics that could be relevant:

```
project = SVS AND issuetype in (Initiative, Epic) AND statusCategory != Done
ORDER BY updated DESC
```

Narrow the results by matching the topic, program area, or components to what the user
described. Then present your best guess:

> "I found **SVS-198 – Q2 BPO Onboarding Initiative** as a likely parent. Does this
> ticket belong there, or is there a different Initiative/Epic it should attach to?"

If nothing clearly matches, tell the user and ask them to specify. Don't leave parent blank
for Tasks and Epics without flagging it.

---

## Step 5: Draft the description

Write in a natural voice — like explaining the work to a teammate, not filling in a form.
Specific language beats vague placeholders: "Migrate the receipt audit workflow to the new
vendor portal" is better than "Complete migration work."

Use this template for all ticket types. Tasks can be shorter; Epics and Initiatives should
be more complete.

```
## Context
[Why does this ticket exist? What problem, opportunity, or ask prompted it?
2–4 sentences. Mention affected users, vendors, systems, or business impact where known.
Use the user's own language where it's clear and natural.]

## Actions to be Completed
- [Specific, concrete step or deliverable]
- [Another step — be as specific as context allows]
- [Add more as needed]

## Conditions of Satisfaction
- [ ] [Verifiable, testable outcome — not "it works" but "the workflow processes
      receipts end-to-end without errors in production"]
- [ ] [Another measurable outcome]
- [ ] [Placeholder if unknown: "Define acceptance criteria with team"]
```

**Writing notes:**
- Context should read like a human wrote it. Vary sentence structure. Use the user's phrasing.
- Actions should be discrete steps a teammate could act on. Avoid "complete analysis" — prefer
  "Pull Q1 receipt audit data and identify top 5 error categories."
- Conditions of Satisfaction must always use checkboxes (`- [ ]`), never plain bullet points —
  even for short or simple tickets. These are outcomes, not activities; frame as things that
  can be checked off when done.

---

## Step 6: Confirm before filing

One compact preview block — not a long form. Show the description draft so the user can
catch tone or content issues before the ticket is created.

> **Ready to file:**
> - **Type:** Epic | **Priority:** P1 | **Due:** May 30, 2026 | **Start:** Apr 21, 2026
> - **Title:** Launch Receipt Audit Workflow for New BPO Partner
> - **Components:** BPO, Workflow Development, Receipt Auditing
> - **Assignee:** Brittney Miller | **Parent:** SVS-198 (Q2 BPO Initiative)
>
> **Description preview:**
> *Context:* We're onboarding a new BPO partner in June and need the receipt audit
> workflow live before their launch date...
> *Actions:* Configure vendor portal access, map existing audit rules, run UAT, document process.
> *Conditions of Satisfaction:* Workflow live in production; BPO team completing audits
> without escalations; SOP published.
>
> Anything to adjust, or shall I go ahead?

Wait for a green light before filing. Apply small tweaks silently. Only re-confirm if
something significant changed (type, assignee, parent).

---

## Step 7: Create the ticket

Use the `createJiraIssue` MCP tool with `contentFormat: "adf"` so mentions and formatting
render correctly in Jira.

**Required fields:**
- `cloudId`: `af51c917-4866-4b81-91f2-4aaa31b02f3e`
- `projectKey`: `SVS`
- `issueTypeName`: Initiative | Epic | Task
- `summary`

**Optional fields:**
- `description` — ADF format, structured per the template above
- `assignee_account_id` — look up via `lookupJiraAccountId` if the user names someone
- `parent` — key of parent Initiative or Epic
- `additional_fields`:
  - `duedate: "YYYY-MM-DD"`
  - `startdate: "YYYY-MM-DD"` (if provided)
  - `components: [{ "name": "AI" }, { "name": "BPO" }]` etc.

**After creating:** Use `createIssueLink` to link any related tickets the user mentioned.

**Always include** `{ "name": "Claude-Generated Ticket" }` in the components array for every
ticket filed through this skill, in addition to any components the user selected.

**Note on priority:** The P0–P4 priority scale may need to be set manually in the Jira UI
after filing if the API rejects the priority field. Let the user know if this happens.

---

## Step 8: Remind about ticket hygiene

Give a brief, friendly reminder about the SVS expectation for active tickets — don't lecture,
just flag it:

> "One thing to keep in mind: SVS best practice is to add a comment at least once a week
> on active tickets — quick summary of progress, blockers, and next steps."

Skip this if the ticket is a small Task that'll likely close quickly.

---

## Step 9: Confirm and close the loop

Reply with:
1. ✅ One-line confirmation with ticket key and title
2. Direct Jira link: `https://instacart.atlassian.net/browse/[KEY]`
3. A prompt to open the ticket and verify it looks right
4. A brief note if anything specific needs checking (priority in UI, parent link, components, etc.)

Always remind the user to open the ticket and review it with a clear, attention-grabbing
alert — not a soft suggestion. Always include the description in the checklist.

Example:
> ✅ Filed: **[SVS-261] Launch Receipt Audit Workflow for New BPO Partner**
> 🔗 https://instacart.atlassian.net/browse/SVS-261
>
> ⚠️ **REMEMBER:** Open the filed ticket and check for accuracy. Especially:
> - **Description** — confirm the context, actions, and conditions of satisfaction read correctly
> - **Priority** — verify P1 is reflected in the Jira UI
> - **Parent** — confirm it's linked to SVS-198 (Q2 BPO Initiative)
> - **Components** — BPO, Workflow Development, Receipt Auditing
>
> Let me know if anything needs fixing!

---

## Error handling

- **Project not found:** Ask the user to confirm the project key (SVS is the default).
- **Invalid work type:** List valid SVS types (Initiative, Epic, Task) and ask.
- **Assignee not found:** File unassigned and note it so they can update in Jira.
- **No parent match found:** Tell the user and ask them to specify or confirm filing without one.
- **MCP failure:** Show exactly what you would have filed so they can do it manually.
