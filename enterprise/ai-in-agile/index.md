---
layout: default
title: Running Your Backlog Through Claude Code or Gemini Code Assist
subtitle: Connect Jira to your coding agent and let your backlog drive implementation.
---

# Running Your Backlog Through Claude Code or Gemini Code Assist

## Connect Jira to your coding agent and let your backlog drive implementation

*A practical guide for developers who want their sprint board to be the trigger for AI-assisted implementation — not a second system they update by hand afterwards. Covers Claude Code and Gemini Code Assist / Gemini CLI side by side.*

*Part of **The AI Software Engineering Playbook** series by [Abdul Bari Shaikh](https://www.linkedin.com/in/abdul-bari-shaikh-etm/).*

> ## What you'll learn
>
> After completing this guide, you'll be able to:
>
> ✔ Connect Jira to Claude Code and/or Gemini Code Assist via MCP
>
> ✔ Ask your agent to read your backlog and propose an implementation order
>
> ✔ Trigger implementation directly from a Jira ticket's description and acceptance criteria
>
> ✔ Have your agent move tickets through your workflow — Not Started → In Progress → In Review/Testing → Done — as work actually happens
>
> ✔ Update or refine user stories without leaving your terminal
>
> ✔ Know where the two agents' mechanics genuinely differ, so switching between them (or running both on the same team) doesn't produce surprises

---

## Why this guide exists

Chapter 1 got your machine ready. Chapter 2 gave you a workflow for turning a decision into a prompt Claude Code can execute autonomously.

There's a gap between them: **where do the decisions come from, and where does the record of what happened go?**

For most teams, that's Jira. But it's common to end up running two disconnected systems — Jira as the source of truth for planning, and Claude Code as a tool you feed prompts into by hand, with someone manually flipping ticket statuses afterwards because they remembered to. Statuses drift from reality. "In Progress" tickets have been sitting untouched for four days. "Done" tickets never got their acceptance criteria checked against what was actually built.

This guide closes that gap: Jira becomes the input **and** the output. You point Claude Code at your backlog, it tells you what's pending, you pick the order, it implements, tests, and updates the ticket status itself — with you approving the parts that matter.

---

**Audience:** Developers and technical leads running sprints with Jira who want their board to reflect real implementation state without manual bookkeeping.

**Estimated time:** 20–30 minutes to connect, ongoing after that.

**Builds on:** Chapter 1 (environment setup) and Chapter 2 (the Chat/Code workflow split). This guide assumes both are already in place.

---

## Prerequisites

- Claude Code and/or Gemini Code Assist (Gemini CLI) installed and authenticated (Chapter 1, Part 3)
- A Jira Cloud site with a project and at least one board
- Node.js (already installed if you followed Chapter 1)
- Optional: permission to create an API token on your Atlassian account (only needed for the auxiliary attachment-handling script in Part 3B, not for the core Jira MCP connection)

---

# Part 1 — Jira access

### 1.1 The core connection needs no token

Part 2's MCP setup uses Jira's **official remote OAuth server**, so for the main workflow in this guide, you don't need to create anything here — you'll authenticate by signing in through a browser when you first connect, the same as any other OAuth login. Skip to Part 1.3.

### 1.2 (Optional) Create an API token for auxiliary scripts

If you later want to script something outside the MCP flow — e.g. downloading a Jira attachment directly for automated processing, which isn't something either agent's Jira MCP tools expose — you'll need a separate API token:

Go to: **https://id.atlassian.com/manage-profile/security/api-tokens**

Click **Create API token**, give it a label such as `jira-scripts`, and copy the token immediately — Atlassian shows it only once.

> Treat this token like a password. It authenticates as *you* — anything run with it will show up in Jira's audit log under your account. This is a separate credential from the MCP OAuth connection; both can coexist.

### 1.3 Confirm your project key and workflow statuses

Open your Jira project and check:
- The **project key** (e.g. `PROJ`, shown as a prefix on every ticket: `PROJ-123`)
- The exact **status names** in your workflow — most teams use something like `To Do`, `In Progress`, `In Review`, `Done`, but yours may differ (`Not Started`, `In Testing`, `Blocked`, and so on)

Status names must match **exactly** when Claude Code transitions tickets later, so write them down as they actually appear on your board.

---

# Part 2 — Connect Jira to your coding agent via MCP

> **A correction, stated plainly:** an earlier draft of this guide specified `npm install -g @modelcontextprotocol/server-jira`. **That package does not exist.** It would fail on `npm install`. The correct approach — for both agents below — is Atlassian's own **official remote MCP server** at `https://mcp.atlassian.com/v1/mcp`, which is OAuth-based and needs no npm package and no API token for this step. (You'll still want an API token from Part 1.1 for one auxiliary script later in this guide — noted where it comes up.)

This guide covers two agents side by side, since teams increasingly mix them. Pick the track for the agent you're using, or read both if you're evaluating.

| | Claude Code | Gemini Code Assist / Gemini CLI |
|---|---|---|
| Config file | `.claude.json` or `.mcp.json` | `~/.gemini/settings.json` (or project `.gemini/settings.json`) — **shared by both the CLI and the IDE agent mode** |
| Add a remote MCP server | `claude mcp add --transport http <name> <url>` | Edit the `mcpServers` block directly (no add subcommand for this) |
| Verify connection | `/mcp` inside a session | `/mcp` inside a session (CLI), or the MCP panel in the IDE for Code Assist |
| Headless/CI invocation | `claude -p "..." --output-format json` | `gemini -p "..." --output-format json` |
| Default write-action behaviour | Prompts for approval per tool call | Prompts for approval per tool call (`--approval-mode default`); `plan` mode is read-only, `yolo` auto-approves everything |

## Track A — Claude Code

### 2A.1 Add the Jira MCP server

```powershell
claude mcp add --transport http jira https://mcp.atlassian.com/v1/mcp
```

Add GitHub the same way if you haven't already, using its official remote server:

```powershell
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

### 2A.2 Authenticate

Restart Claude Code, then run:

```
/mcp
```

Both servers should show as needing authentication on first connect — select each one and Claude Code opens a browser window for the OAuth consent screen. Approve it, and the token is stored for subsequent sessions.

Expected output after authenticating:
```
github · ✔ connected · N tools
jira   · ✔ connected · N tools
```

If a server shows `✘ Failed to connect` or a repeated OAuth redirect loop, run `claude mcp get jira` to see the auth URL directly and complete the flow manually, rather than assuming the CLI command alone did it — this is the same "don't trust a status without raw evidence" principle from the rest of this series.

### 2A.3 Test it can actually read your board

Ask Claude Code, inside a session:

```
List the open tickets in project PROJ, grouped by status.
```

A response listing real tickets from your board — not a generic "I don't have access" reply — confirms the connection is live.

## Track B — Gemini Code Assist / Gemini CLI

### 2B.1 Add the Jira MCP server

Open (or create) `~/.gemini/settings.json`:

```powershell
code $env:USERPROFILE\.gemini\settings.json
```

Add the `mcpServers` block. Because Atlassian's remote server supports OAuth discovery, you only need the URL — no token, no headers:

```json
{
  "mcpServers": {
    "jira": {
      "httpUrl": "https://mcp.atlassian.com/v1/mcp"
    },
    "github": {
      "httpUrl": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

This same file is read by **both** Gemini CLI in the terminal and Gemini Code Assist's agent mode in VS Code or IntelliJ — one edit covers both.

### 2B.2 Authenticate and verify

If you're using the terminal, restart Gemini CLI and run:

```
/mcp
```

You should see each server listed as connected, with its tool count — for example `🟢 jira - Ready (N tools)`. If you're using Gemini Code Assist in VS Code instead, open the agent mode panel and check the same server list there; the connection state is shared, but the panel is where IDE-side confirmation happens.

If tools don't appear, run `/mcp reload` to force a re-query, or check that the OAuth flow actually completed rather than assuming a "connected" label means the same thing as tools being available — the same distinction that tripped up the Jira connector earlier in this series.

### 2B.3 Test it can actually read your board

```
gemini -p "List the open tickets in project PROJ, grouped by status."
```

or, inside an interactive session, just ask the same thing conversationally.

---

# Part 3 — The Workflow: Backlog In, Working Code Out

This is the core loop the guide is built around. It has four stages, and your agent — Claude Code or Gemini Code Assist, the prompts below work for either — drives all of them from inside one session. Where the two agents' mechanics genuinely diverge (mainly around headless invocation and approval flags), that's called out explicitly rather than papered over.

### Stage 1 — List what's pending

```
Read all tickets in project PROJ with status "To Do" or "Blocked".
For each one, show: key, summary, and acceptance criteria.
Do not start implementing anything yet.
```

Your agent reads the board and reports back — this is a read-only step, safe to run anytime, even mid-sprint, to get an honest picture of what's actually outstanding versus what people remember being outstanding.

> **Gemini-specific option:** run this stage with `--approval-mode plan`, Gemini's built-in read-only mode. It structurally can't call a write tool regardless of what the prompt says, which is a stronger guarantee than "the prompt said don't implement anything yet" — worth using for this stage specifically, on either agent's underlying discipline: constrain by tool availability where you can, not just by instruction.

### Stage 2 — You choose the order

This is the one decision your agent should not make for you — priority is a product and business call, not a code call. Review the list, then tell it explicitly:

```
Implement these in this order: PROJ-142, PROJ-139, PROJ-145.
Skip PROJ-140 for now — it's blocked on a design decision.
```

If you genuinely want help sequencing, ask it to *propose* an order and give reasons (dependencies, size, risk) — but treat that as input to your decision, not the decision itself, the same separation Chapter 2 draws between the planning role and the execution role.

### Stage 3 — Implement, transitioning status as it goes

This is where the ticket lifecycle and the code lifecycle become the same event. Give Claude Code an instruction like:

```
# Context
# Project: MyApp
# Working folder: D:\projects\myapp
# Python: .venv\Scripts\python.exe
# Do NOT touch: src/auth/, config/database.py

For PROJ-142:
1. Transition the ticket to "In Progress".
2. Read the ticket's description and acceptance criteria.
3. Read the relevant existing code before making changes.
4. Implement the change described.
5. Run: python -m pytest tests/ -v
   All existing tests must still pass, plus any new test for this ticket.
6. If tests pass: transition the ticket to "In Review/Testing",
   add a comment on the ticket summarising what changed and which
   tests were run, and commit with message "[PROJ-142] <description>".
7. If tests fail after 3 attempts: leave the ticket in "In Progress",
   add a comment explaining what's failing, and stop.

Then repeat the same sequence for PROJ-139, then PROJ-145.
```

Notice this is Chapter 2's iteration-loop template with two additions: a **status transition** at the start and end of each ticket, and a **ticket comment** that records what actually happened — so the audit trail lives in Jira, not just in your terminal scrollback.

**Running this unattended (Gemini):** the same instruction block works as a headless invocation for CI or scheduled runs:

```powershell
gemini -p "<the instruction block above>" --approval-mode auto_edit --output-format json
```

`auto_edit` auto-approves file edits (so the loop doesn't stall waiting for a keypress that will never come) while still prompting-and-failing-safe on anything outside that — a middle ground between `default` (blocks unattended runs entirely) and `--yolo` (auto-approves *everything*, including the ticket transitions and git commit, which is more autonomy than Stage 4 below says you should actually want).

**Claude Code's headless equivalent** is the same idea with a different flag surface: `claude -p "<instruction>" --output-format json`, scoped down with `--allowedTools` to the specific tools this loop needs (Bash, Edit, Read, and the Jira/GitHub MCP tools) rather than granting blanket access — Claude Code doesn't have a single named "auto_edit" mode, so the equivalent restraint comes from an explicit allowlist instead.

### Stage 4 — You close the loop

Deliberately leave the final `Done` transition to you, or at minimum to a separate explicit instruction:

```
Show me the diff and test output for PROJ-142 before you mark it Done.
```

Moving a ticket to "In Review/Testing" is a reasonable autonomous action — it's saying "I believe this is finished, please look." Moving it all the way to "Done" is a claim that the work has been *accepted*, which is a human judgement, not something the same agent that wrote the code should certify unilaterally. Keep that one gate manual, even if everything else in the loop runs on its own — and on Gemini specifically, that means this step should never run under `--yolo`, regardless of how tempting full automation looks once the rest of the loop is working smoothly.

---

## Four Practices Specific to Backlog-Driven Development

### Practice 1 — Never let your agent invent a ticket's scope

If a ticket's acceptance criteria are vague or missing, that is a signal to stop and ask, not to guess and proceed:

```
PROJ-145's acceptance criteria are empty. Don't implement anything yet —
summarise what you think the ticket is asking for, and I'll confirm or correct it.
```

An agent that fills in ambiguous scope with a reasonable-sounding guess will confidently build the wrong thing just as fast as it builds the right one.

### Practice 2 — Match ticket status names exactly, and confirm the transition worked

Jira workflow transitions are name-sensitive and sometimes conditional (some transitions require fields to be filled, or are only available from certain prior statuses). Always have your agent confirm the transition succeeded rather than assume it:

```
After transitioning PROJ-142, re-read the ticket and confirm its status
now shows "In Review/Testing". If the transition failed, tell me why.
```

### Practice 3 — Keep the Jira comment honest, not celebratory

Ask for the comment to state what was run and what the result was, not a summary that reads like a status report:

```
The comment on the ticket should state exactly which test command was run
and its literal pass/fail output — not a general "implementation complete" statement.
```

This is the same discipline as Chapter 2's exact-verification practice, just aimed at the artefact that other people on your team will actually read.

### Practice 4 — Use Jira to catch drift, not just to record it

Periodically run the read-only Stage 1 query against `In Progress` tickets specifically:

```
List every ticket currently in "In Progress". For each, check the last
commit or comment timestamp associated with it.
```

A ticket that's been "In Progress" for six days with no associated activity is a planning problem hiding behind a status label — this query surfaces it before standup does.

---

## The full copy-paste template

```
# Context
# Project: [name]
# Working folder: [full path]
# Python: [path to python]
# Frozen — do NOT modify: [list files and folders]

## Ticket
[TICKET-KEY]

## Step 1 — Transition and read
Transition [TICKET-KEY] to "[in-progress status name]".
Read the ticket's description and acceptance criteria.
Read [relevant existing file(s)] before making any change.

## Step 2 — Implement
[What to change, based on the ticket's acceptance criteria.]

## Step 3 — Verify
Run: [test command]
Expected: [pass condition]
Try up to [N] iterations. Stop early once the target is met.

## Step 4 — Report and transition
If verification passes:
  Transition [TICKET-KEY] to "[review status name]".
  Add a ticket comment stating the exact test command and its literal output.
  Commit: git commit -m "[TICKET-KEY] description"
If verification still fails after [N] attempts:
  Leave status as "[in-progress status name]".
  Add a comment explaining what's failing.
  Stop and report to me.

## Constraints — do NOT touch:
- [file or folder]

## Do NOT do without asking:
- Transition this ticket to "[done status name]"
```

---

## Before and after

| | Backlog and code kept separately | Backlog-driven agent workflow |
|---|---|---|
| Ticket status accuracy | Manually updated, often stale | Updated at the moment work actually happens |
| Implementation order | Decided, then re-explained per prompt | Decided once, executed straight through |
| Audit trail | Scattered across chat history | Lives on the ticket itself, as comments |
| Standup prep | "Let me check what's actually done" | Board reflects reality already |
| Risk of scope drift | Ticket says one thing, code does another | Agent stops and asks when criteria are unclear |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Jira MCP shows "connected" but has zero tools | Connection and authorization are two separate states — re-run `/mcp` (Claude Code) or `/mcp reload` (Gemini) and actually complete the OAuth screen rather than trusting the label alone |
| OAuth browser tab never returns / hangs | Confirm you're already logged into the relevant web account in your default browser before starting the connect flow — a mismatched or signed-out browser session is the most common cause |
| Transition appears to succeed but status doesn't change | Status name mismatch — copy the exact name from the board, or the transition may require a field your agent didn't set |
| Agent implements a ticket differently than expected | Acceptance criteria were vague — add Practice 1's confirm-scope-first step |
| Ticket stuck "In Progress" with no visible reason | Ask your agent to check for a blocking test failure it may not have surfaced clearly — run Practice 4's drift check |
| Comments on tickets read like marketing copy, not evidence | Re-state Practice 3's instruction explicitly in the prompt each time until it's habitual |
| Gemini: tools not appearing after editing `settings.json` | Run `/mcp reload` inside a session to force re-discovery, rather than assuming a restart alone picked up the change |
| Gemini: headless run stalls waiting for approval that never comes | You're on `--approval-mode default` in a non-interactive context — use `auto_edit` for the implementation loop, never `yolo` for the Stage 4 Done transition |
| Gemini: `Read` tool / image-viewing behaves inconsistently on an attachment | Known, still-open reliability gap in image-file handling by path in some agent CLIs' file-reading tools — for anything unattended, send the image to the model API directly (base64, vision input) rather than relying on file-path reads |

---

*For more guides like this — follow [Abdul Bari Shaikh on LinkedIn](https://www.linkedin.com/in/abdul-bari-shaikh-etm/) and turn on notifications.*

---

This guide is part of **The AI Software Engineering Playbook** — a growing collection of practical engineering patterns for AI-assisted software development.

Additional Foundation, Application Development, and Enterprise guides will be published over time.

[Back to the Playbook](../../)
