---
layout: default
title: Stop micro-managing your AI
---

# Still Coding Traditionally in the AI Era? Stop Micro-managing Your AI.

*A beginner-friendly guide for anyone who wants AI to do the heavy lifting — properly.*

*Part of the [Ship It with Claude Code](../) series by [Abdul Bari Shaikh](https://www.linkedin.com/in/abdul-bari-shaikh-etm/)*

---

## Sound familiar?

You ask AI to write some code. It looks good. You run it. It breaks. You paste the error back. It fixes one thing, breaks another. You fix that. Now you have forgotten what you changed three messages ago. The AI has forgotten too.

Forty-five minutes later you have had twelve conversations, your tab bar is a disaster, and you still do not have working code.

**This is not how it has to work.**

There is a different way — one where you write the goal once, Claude iterates autonomously until it succeeds, runs its own tests, commits the result, and hands back to you when done.

---

## Two tools, two jobs — never mix them up

Most people use Claude Chat and Claude Code interchangeably. That is the mistake.

They have completely different jobs and you should never ask one to do the other's work.

**Claude Chat is your architect.**
Use it to brainstorm, prioritise, and make decisions before a single line of code is written. Ask it to help you think through features for the current sprint, weigh architectural options, and understand trade-offs. When you have made a decision, ask Claude Chat to write the Claude Code prompt for you — in the structured instructional format this guide teaches.

**Claude Code is your code assistant.**
It receives that prompt, reads your actual files, writes and runs the code, iterates until the tests pass, and commits the result. It does not make architectural decisions. It executes them.

**The rule that makes this work:**

> Never ask Claude Chat to fix a code error.
> Never ask Claude Code to make an architectural decision.

When something breaks, do not paste the stack trace into Claude Chat. Write a Claude Code prompt that describes what is failing and what success looks like, and let Claude Code fix it autonomously.

When you are unsure whether to use a REST API or GraphQL, do not ask Claude Code. Open Claude Chat, explain your constraints, and let it help you decide. Then ask it to write the Claude Code prompt that implements the decision.

**A typical sprint workflow looks like this:**

1. Open Claude Chat — brainstorm the features for this sprint
2. Claude Chat helps you prioritise and make the architectural call
3. Claude Chat writes the Claude Code prompt for the first feature
4. Paste that prompt into Claude Code — it reads, writes, tests, commits
5. Return to Claude Chat for the next feature or decision
6. Repeat

The result: Claude Chat keeps the big picture. Claude Code handles the details. You stay in control of both without getting lost in either.

---

## What Claude Code and MCP actually are

**Claude Code** is Claude running directly in your terminal or IDE — not a chat window, but an agent with real access to your files, your tests, and your git history.

**MCP (Model Context Protocol)** are integrations that extend what Claude Code can reach — your GitHub issues, your database schema, your API docs. Think of them as giving Claude a full desk setup instead of a sticky note.

The result: an autonomous coding agent that iterates until the job is done.

---

## The 7 habits that make it work

---

### Habit 1 — Give Claude a map before it touches anything

The single biggest source of AI coding mess is Claude not knowing where it is. Fix this in 30 seconds with a context block at the top of every prompt:

```
# Project: MyApp
# Working folder: D:\projects\myapp
# Python: .venv\Scripts\python.exe
# Do NOT touch: src/auth/, config/database.py
# Branch: feature/payments
```

This is not over-engineering. It is the difference between Claude editing the right file and Claude editing a file that happens to have a similar name.

---

### Habit 2 — Make Claude read before it writes

The most common Claude Code failure: it edits code it has not actually read. Add read first as an explicit step:

```
## Step 1 — Read before touching
Read src/payments/processor.py
Find the function that handles refunds.

## Step 2 — Only then make the change
Add a minimum amount check before any refund is processed.
```

Claude edits based on what the file actually contains right now — not how it looked when you last pasted it into a chat window.

---

### Habit 3 — Tell Claude exactly what success looks like

Vague: *make sure it works*

Specific:

```
Run: python -m pytest tests/test_payments.py -v
Expected output must include: "5 passed"
If any test fails, stop and report — do not proceed.
```

The second version gives Claude a pass/fail signal it can act on without asking you. This is what makes autonomous iteration possible.

---

### Habit 4 — Name what Claude must not touch

You have probably had AI helpfully refactor something you did not ask about. Stop this permanently:

```
## Constraints — do NOT modify:
- src/auth/
- config/database.py
- any file in tests/
- requirements.txt
```

---

### Habit 5 — One change, one commit, one message

Ask Claude to commit after each fix in a specific format:

```
git add src/payments/processor.py
git commit -m "fix(payments): add minimum refund check"
```

Clean git history automatically. When something breaks three days from now you can trace it to one commit with one purpose.

---

### Habit 6 — Verify at every step, not just the end

Do not save all testing for the last step. Verify after every meaningful change:

```
## Step 2 — Add the endpoint
Edit src/api/routes.py

Verify immediately:
  python -c "from src.api.routes import router; print('OK')"

## Step 3 — Write the test
python -m pytest tests/test_new_endpoint.py -v
Expected: 1 passed
```

Catching a broken import in step 2 takes five seconds. Catching it after five more steps takes twenty minutes of archaeology.

---

### Habit 7 — Let Claude iterate autonomously until it gets it right

**This is the habit that changes everything.**

Some tasks cannot be solved in a single pass. Instead of sending follow-up messages for each iteration, write the loop directly into the prompt:

```
## Goal
Get payment validation passing all tests.
Try up to 3 iterations. Stop the moment all tests pass.

## Iteration loop
1. Run: python -m pytest tests/test_payments.py -v

2. All tests pass — stop, go to Final Steps.
   Still failing AND attempts less than 3 — fix the specific failure, repeat.
   Attempts reach 3 — stop, report what failed, go to Final Steps.

3. Network error or rate limit — pause and report.
   Do NOT count that as an attempt.

## Final Steps — run these regardless of outcome
python -m pytest tests/ -q
git commit -m "fix(payments): validation loop"
Report what passed and what still needs attention.
```

Claude diagnoses each failure, makes a targeted fix, and re-runs on its own. The context never gets lost because Claude Code is working directly in your project the whole time.

On a real project this week Claude ran two iterations, hit the target, committed the result. One message from me. Zero follow-ups.

---

## The full copy-paste template

```
# Context
# Project: [name]
# Working folder: [full path]
# Python: [path to python]
# Frozen — do NOT modify: [list files and folders]

## Goal
[What does done look like?]
Target: [measurable outcome — test count, score, output]
Try up to [N] iterations. Stop early if target is reached.

## Step 1 — Read first
Read [file] and find [function or section].

## Step 2 — Make the change
[Exactly what to change and where.]

Verify after editing:
  [smoke test command]

## Iteration loop
1. Run: [test command]
2. Target met — go to Final Steps.
   Not met AND attempts less than [N] — fix the failure and repeat.
   Attempts reach [N] — report status and go to Final Steps.
3. External failure — pause, report, do not count as attempt.

## Final Steps
Run full suite: [test command]
Must show: [expected result]
Commit: git commit -m "[type(scope): description]"

## Constraints — do NOT touch:
- [file or folder]
```

---

## Before and after

| | Traditional AI coding | Agent-first approach |
|---|---|---|
| Messages sent | 10 to 15 | 1 |
| Context lost | Every session | Never |
| Architectural decisions | Mixed into code chat | Claude Chat, separately |
| Git history | One big blob | Clean traceable commits |
| Time to working code | 45 plus minutes | Minutes |
| What you do | Paste errors, repeat | Come back to passing tests |

---

## Quick reference card

| Habit | Problem it solves |
|---|---|
| Claude Chat for architecture | Mixing decisions and code causes both to suffer |
| Claude Code for execution | Chat cannot see your files or run your tests |
| Context block | Wrong files, wrong branch |
| Read first | Edits that miss what the code looks like today |
| Exact verification | Assuming success when tests are failing |
| Constraints list | Unwanted changes to unrelated files |
| One commit per fix | Untraceable git history |
| Verify each step | Cascading failures from a silent break |
| Iteration loop | Endless back-and-forth, lost context every session |

---

*For more guides like this — follow [Abdul Bari Shaikh on LinkedIn](https://www.linkedin.com/in/abdul-bari-shaikh-etm/) and turn on notifications.*

*Next guide: [MERN Stack with Claude Code](../mern-stack/) — coming soon.*

*[Back to all guides](../)*
