---
layout: default
title: AI-assisted Development Environment
subtitle: Setting up a modern AI software engineering workstation on Windows.
---

# Development Environment

## Setting up a modern AI software engineering workstation on Windows

*A practical guide for developers who want to build a reliable AI-assisted development environment.*

*Part of **The AI Software Engineering Playbook** series by [Abdul Bari Shaikh](https://www.linkedin.com/in/abdul-bari-shaikh-etm/).*

> ## What you'll learn
>
> After completing this guide, you'll be able to:
>
> ✔ Configure a complete AI-assisted development environment on Windows
>
> ✔ Install and integrate Python, VS Code, Git, GitHub CLI and Node.js
>
> ✔ Set up Claude Code with GitHub MCP
>
> ✔ Configure projects for efficient AI-assisted software development
>
> ✔ Verify and troubleshoot your development environment

---

## Why this guide exists

Modern AI can generate code, but productive software engineering requires much more than a coding assistant.

Developers need an environment where AI can understand the project, access source control, execute commands, run tests, and work within established engineering workflows.

This guide provides that foundation by walking through the complete setup of a modern AI-assisted development environment on Windows.

---

**Audience:** Developers and software engineers setting up a Windows workstation for AI-assisted software development

**Estimated time:** 30–45 minutes

**Tested on:** Windows 10, Windows 11 and PowerShell 5+

---

## Prerequisites

- Windows 10 (version 1809+) or Windows 11
- Administrator access to install software
- An Anthropic account with a Claude Pro, Max, Team or Enterprise subscription
- A GitHub account

---

# Part 1 — Core Development Tools

## 1.1 Install Python

Download the latest Python 3.11 or later from:

https://www.python.org/downloads/

Run the installer and ensure the following options are selected:
- ✅ **Add Python to PATH** (critical — check this before clicking Install)
- ✅ Install for all users (recommended)

Verify in a new PowerShell window:
```powershell
python --version
pip --version
```

### 1.2 Install VS Code

Download from https://code.visualstudio.com/download — choose the **System Installer** (not User Installer) for Windows.

During install check:
- ✅ Add to PATH
- ✅ Register Code as editor for supported file types
- ✅ Add "Open with Code" to Explorer context menu

Verify:
```powershell
code --version
```

Install essential VS Code extensions (run in PowerShell):
```powershell
code --install-extension ms-python.python
code --install-extension ms-python.venv
code --install-extension GitHub.copilot
code --install-extension eamodio.gitlens
```

### 1.3 Install Git for Windows

Download from https://git-scm.com/download/win

During install, key options:
- Default editor: choose **Visual Studio Code**
- ✅ Add Git to PATH (default — keep it)
- Line ending: **Checkout Windows-style, commit Unix-style** (recommended)

Verify:
```powershell
git --version
```

Configure your identity (used in all commits):
```powershell
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

### 1.4 Install Node.js

Download LTS version from https://nodejs.org/en/download

Run the installer with default options. Node.js is required for Claude Code and MCP servers.

Verify:
```powershell
node --version   # should be 18+
npm --version
```

### 1.5 Install GitHub CLI

Download the Windows installer from https://cli.github.com/

Or install via winget (available on Windows 11 and updated Windows 10):
```powershell
winget install --id GitHub.cli
```

Verify:
```powershell
gh --version
```

---

## Part 2 — GitHub Authentication

### 2.1 Create a Personal Access Token (Classic)

Go to: **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**

Click **Generate new token (classic)**

Set:
- Note: `claude-code-dev` (or any label you recognise)
- Expiration: 90 days (or No expiration for dev machines)
- Scopes — check these:
  - ✅ `repo` (full — all sub-items)
  - ✅ `read:org`
  - ✅ `read:user`
  - ✅ `gist`

Click **Generate token** and **copy it immediately** — GitHub shows it only once.

> **Save this token** in a password manager or secure note. You will need it in Part 4 for the MCP server. It starts with `ghp_`.

### 2.2 Authenticate GitHub CLI

Run in PowerShell:
```powershell
gh auth login
```

You will be asked a series of questions — answer as follows:
```
Where do you use GitHub?          → GitHub.com
What is your preferred protocol?  → HTTPS
Authenticate Git with credentials? → Yes
How would you like to authenticate? → Login with a web browser
```

GitHub CLI will display an **8-digit one-time code** like `ABCD-1234` and open your browser automatically.

In the browser:
1. Go to https://github.com/login/device (opens automatically)
2. Enter the 8-digit code shown in your terminal
3. Click **Authorise GitHub CLI**
4. You may be asked to confirm with your GitHub password

Back in the terminal you will see:
```
✓ Authentication complete.
✓ Logged in as YourUsername
```

Verify:
```powershell
gh auth status
```

Expected output:
```
github.com
  ✓ Logged in to github.com account YourUsername
  - Active account: true
  - Token scopes: gist, read:org, read:user, repo
```

---

## Part 3 — Claude Code

### 3.1 Install Claude Code

Install via npm (requires Node.js from Part 1.4):

```powershell
npm install -g @anthropic-ai/claude-code
```

Close PowerShell and open a new window so PATH updates take effect.

Verify:
```powershell
claude --version
```

> If `claude` is not recognised after reopening, find npm's global bin folder and add it to PATH:
> ```powershell
> npm config get prefix
> ```
> Copy the path it returns, then go to **Start → Search "Environment Variables" → System Properties → Environment Variables → Path → Edit → New** and paste it. Open a new PowerShell window and try again.

> **Alternative — native installer** (no Node.js required, auto-updates):
> ```powershell
> irm https://claude.ai/install.ps1 | iex
> ```
> Both methods install the same Claude Code. Use npm if you want version control; use the native installer if you want zero dependencies.

### 3.2 Authenticate Claude Code

Run in PowerShell from any directory:

```powershell
claude
```

On first launch, Claude Code opens your browser automatically and asks you to sign in to your Anthropic account (the same account as claude.ai).

Sign in with your Anthropic credentials. The browser will show a confirmation page. Return to the terminal — authentication completes automatically.

You should see the Claude Code prompt:
```
▐▛███▜▌   Claude Code v2.x.xxx
▝▜█████▛▘  Sonnet 4.6 · Claude Pro
```

Type `exit` to leave Claude Code for now and continue with the setup.

---

## Part 4 — Set Up Your Project

You have two starting points depending on whether the project already exists on GitHub or you are starting fresh.

---

### Path A — Clone an existing GitHub repo

Open PowerShell and run:

```powershell
cd D:\projects
gh repo clone YourUsername/YourRepoName
cd YourRepoName
code .
```

VS Code opens with the project loaded. Continue to **Step 4.3**.

---

### Path B — Create a new project from scratch

Open **Windows Explorer**, navigate to `D:\projects`, click the address bar, type `cmd` and press Enter. This opens a Command Prompt already inside that folder.

```cmd
mkdir MyProject
cd MyProject
code .
```

VS Code opens with the empty project folder loaded.

**Initialise git and connect to GitHub:**

In the VS Code terminal (**Ctrl+`**):

```powershell
git init
git branch -M main
gh repo create MyProject --private --source=. --push
```

This creates the private repo on GitHub and pushes the initial commit in one step.

---

### 4.3 Select Python interpreter

In VS Code press **Ctrl+Shift+P**, type `Python: Select Interpreter` and press Enter. Choose the Python version you installed in Part 1.1 (e.g. `Python 3.11.x`).

### 4.4 Create virtual environment

In the VS Code terminal:

```powershell
python -m venv .venv
.venv\Scripts\activate
```

Your prompt should now show `(.venv)`:
```
(.venv) PS D:\projects\MyProject>
```

### 4.5 Create `requirements.txt` and install dependencies

Create `requirements.txt` in your project root with your packages — sample starting point:

```
fastapi==0.115.0
requests==2.32.3
python-dotenv==1.0.1
```

Then install:

```powershell
pip install -r requirements.txt
```

### 4.6 Create `CLAUDE.md`

Claude Code reads this file automatically at every session start — it is its permanent memory about your project.

```powershell
code CLAUDE.md
```

Minimum contents:

```markdown
# Project Name

## What this is
Brief description. Stack: Python, ...

## Project structure
- src/          — core source code
- config/       — configuration

## Common commands
pip install -r requirements.txt

## Phase status
Phase 1: COMPLETE
Phase 2: IN PROGRESS
```

### 4.7 Launch Claude Code in your project

```powershell
claude
```

Claude Code now starts with full context of your project, reads `CLAUDE.md` automatically, and is ready for development.

---

## Part 5 — GitHub MCP Server for Claude Code

The MCP server lets Claude Code read issues, create PRs, check commits, and interact with your GitHub repo directly from the terminal.

### 5.1 Install the GitHub MCP package globally

```powershell
npm install -g @modelcontextprotocol/server-github
```

> This package was deprecated by its maintainer in April 2025 but remains fully functional for connecting Claude Code to GitHub via the standard REST API. It requires only your classic PAT — no GitHub Copilot subscription needed.
>
> **Do not use** `https://api.githubcopilot.com/mcp` as an alternative — that endpoint requires an active GitHub Copilot licence and returns a 401 unauthorised error without one.

### 5.2 Add GitHub MCP to Claude Code config

The `claude mcp add` CLI has known Windows parsing bugs with `npx -y` and JSON flags. The most reliable method is to edit the config file directly.

**Open the config file in VS Code:**
```powershell
code $env:USERPROFILE\.claude.json
```

Find the end of the file (just before the final closing `}`) and add the `mcpServers` block. If a `mcpServers` key already exists, add the `github` entry inside it. If it does not exist, add the whole block:

```json
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_xxxx"
      }
    }
  }
```

Replace `ghp_xxxx` with the classic PAT you created in Part 2.1.

> The `cmd /c` wrapper is required on Windows — it routes the npx call through cmd.exe, bypassing the PowerShell argument parsing bug that causes `error: unknown option '-y'`.

**Validate the JSON before saving:**

Use **Ctrl+Shift+P → Format Document** in VS Code. If VS Code reports a JSON syntax error, fix it before saving (usually a missing comma or mismatched bracket).

### 5.3 Verify MCP connection

Restart Claude Code, then run inside the Claude Code session:
```
/mcp
```

Expected output:
```
github · ✔ connected · 26 tools
```

If it shows `✘ Failed to connect`, check:
1. Your PAT has `repo`, `read:org`, `read:user` scopes
2. The PAT has not expired
3. `npm list -g @modelcontextprotocol/server-github` confirms global install

### 5.4 Test GitHub MCP is reading your repo

Ask Claude Code:
```
Show me the open issues in my repo
```

A response listing issues (or confirming none exist) means everything is working.

---

## Part 6 — Personal Access Token for Kaggle Notebooks

When you move work to a Kaggle notebook (for GPU access), you need the same GitHub PAT to clone your private repo.

**Add the PAT as a Kaggle Secret:**

1. Go to https://www.kaggle.com/settings/connected-accounts — verify GitHub is connected
2. Open any Kaggle notebook → **Add-ons → Secrets → Add Secret**
3. Name: `GITHUB_TOKEN`
4. Value: your `ghp_xxxx` PAT

In the notebook, access it and clone your repo:

```python
from kaggle_secrets import UserSecretsClient

# Retrieve the token
token = UserSecretsClient().get_secret("GITHUB_TOKEN")

# Clone using the authenticated HTTPS URL
import os
os.system(f"git clone https://{token}@github.com/YourUsername/YourRepoName.git")
```

> `os.system()` is the simplest method in Kaggle notebooks. It runs the git command directly in the shell — no additional imports or configuration needed.

> Use the same classic PAT with `repo` scope — it works identically from Kaggle.

---

## Quick Reference — Daily Commands

```powershell
# Start a session
cd D:\projects\YourProject
.venv\Scripts\activate
claude

# Inside Claude Code
/mcp                          # check MCP server connections
/status                       # confirm CLAUDE.md loaded, git branch

# Git via GitHub CLI
gh repo view                  # repo summary
gh issue list                 # open issues
gh pr list                    # open pull requests
gh pr create --title "title"  # create PR for current branch

# Common git
git status
git add .
git commit -m "[PHASE2] description"
git push
git pull
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `claude` not recognised after install | Open new PowerShell window; PATH updates require restart |
| `error: unknown option '-y'` in mcp add | Edit `~\.claude.json` directly using the cmd wrapper approach in Part 5.2 |
| GitHub MCP `✘ Failed to connect` | Check PAT scopes (`repo`, `read:org`, `read:user`) and expiry |
| `api.githubcopilot.com` returns 401 | That endpoint requires Copilot subscription — use the npm package approach instead |
| `(.venv)` not showing in prompt | Run `.venv\Scripts\activate` from project root |
| Claude Code auth loop | Run `claude logout` then `claude` again to re-authenticate |

---

*For more guides like this — follow [Abdul Bari Shaikh on LinkedIn](https://www.linkedin.com/in/abdul-bari-shaikh-etm/) and turn on notifications.*

---

This guide is part of **The AI Software Engineering Playbook** — a growing collection of practical engineering patterns for AI-assisted software development.

Additional Foundation, Application Development, and Enterprise guides will be published over time.

[Back to the Playbook](../../)
