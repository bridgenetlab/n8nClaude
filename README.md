# n8n Workflow Builder with Claude Code

An AI-powered workspace for building production-ready n8n workflows. Claude acts as your
n8n architect — you describe the automation, Claude designs and builds it directly inside
your n8n instance using the n8n MCP server and domain-specific skills.

**What this setup enables:**
- Build complete n8n workflows by describing them in plain language
- Claude validates, tests, and documents every workflow before declaring it done
- 20 MCP tools give Claude direct read/write access to your n8n instance
- 7 domain skills inject expert n8n knowledge into every session automatically

---

## How It Works

```
You describe a workflow
        │
        ▼
Claude asks clarifying questions
(trigger type, data sources, error handling, credentials)
        │
        ▼
Claude checks existing workflows + searches 2,709 templates
        │
        ▼
Claude plans the workflow and presents it for your approval
        │
        ▼
Claude builds it live in your n8n instance via MCP tools
        │
        ▼
Claude validates → tests → adds sticky notes + description
        │
        ▼
Workflow is ready in n8n
```

Two tools power this:
- **[n8n-mcp](https://github.com/czlonkowski/n8n-mcp)** — gives Claude direct API access to your n8n instance (create, update, validate, test workflows)
- **[n8n-skills](https://github.com/czlonkowski/n8n-skills)** — injects expert n8n knowledge into Claude's context (expressions, node config, patterns, validation)

---

## Repository Contents

| File | Purpose |
|---|---|
| `CLAUDE.md` | Claude's persistent instructions — role, tools, quality standards, workflow creation process |
| `.mcp.json` | MCP server config — connects Claude to your n8n instance via the n8n API |
| `README.md` | This file |

> **Note for collaborators:** `.mcp.json` in this repo is configured for the owner's n8n instance.
> You must update `N8N_API_URL` and `N8N_API_KEY` with your own values before use (see Step 3 below).

---

## Prerequisites

### 1. Node.js v18 or later

Node.js provides `npx`, which is used to run the n8n-mcp server automatically.

**Windows**
```powershell
# Option A — using winget (recommended)
winget install OpenJS.NodeJS.LTS

# Option B — download installer from https://nodejs.org and run it
```

**macOS**
```bash
# Using Homebrew (install Homebrew first at https://brew.sh if needed)
brew install node
```

**Linux (Debian/Ubuntu)**
```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
```

**Verify installation**
```bash
node --version   # should print v18.x.x or higher
npx --version    # should print 9.x.x or higher
```

---

### 2. VS Code + Claude Code Extension

Claude Code runs as a VS Code extension that adds an AI coding assistant panel to your editor.

**Install VS Code** (skip if already installed)
- Download from [https://code.visualstudio.com](https://code.visualstudio.com) and run the installer

**Install the Claude Code extension**

1. Open VS Code
2. Press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (macOS) to open Extensions
3. Search for **Claude Code**
4. Click **Install** on the extension published by **Anthropic**

Or install directly from the terminal:
```bash
code --install-extension anthropic.claude-code
```

**Sign in**
1. After installing, click the Claude Code icon in the VS Code Activity Bar (left sidebar)
2. Click **Sign In** and authenticate with your Anthropic account

---

### 3. Git

Git is required to clone this repository and to install n8n-skills.

**Windows**
```powershell
winget install Git.Git
# After install, restart your terminal
```

**macOS**
```bash
brew install git
# or: xcode-select --install (installs git as part of Xcode tools)
```

**Linux**
```bash
sudo apt-get install git   # Debian/Ubuntu
sudo dnf install git       # Fedora/RHEL
```

**Verify**
```bash
git --version
```

---

### 4. An n8n Instance

You need a running n8n instance that Claude can connect to.

**Option A — n8n Cloud (recommended, no setup needed)**
1. Sign up at [https://app.n8n.cloud](https://app.n8n.cloud)
2. Your instance URL will be something like `https://yourname.app.n8n.cloud`

**Option B — Self-hosted with npx**
```bash
npx n8n
# n8n starts at http://localhost:5678
```

**Option C — Self-hosted with Docker**
```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
# n8n starts at http://localhost:5678
```

---

### 5. n8n API Key

Claude needs an API key to connect to your n8n instance.

1. Open your n8n instance in the browser
2. Click your avatar (bottom-left) → **Settings**
3. Go to **API** in the left menu
4. Click **Create API Key**
5. Give it a name (e.g. `claude-code`)
6. Copy the key — you will need it in Step 3 of the setup

---

## Setup

### Step 1: Clone This Repository

```bash
git clone <your-repo-url>
cd n8n_workflow
```

---

### Step 2: Open the Folder in VS Code

```bash
code .
```

Or from VS Code: **File → Open Folder** → select the `n8n_workflow` folder.

---

### Step 3: Configure `.mcp.json`

Open `.mcp.json` in VS Code and update the two values for your instance:

```json
{
  "mcpServers": {
    "n8n": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_API_URL": "https://your-instance.app.n8n.cloud",
        "N8N_API_KEY": "your-api-key-here",
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true"
      }
    }
  }
}
```

| Field | Value |
|---|---|
| `N8N_API_URL` | Root URL of your n8n instance, no trailing slash |
| `N8N_API_KEY` | The API key you created in Prerequisite 5 |

Save the file. The MCP server will use these values the next time you start a Claude session.

---

### Step 4: Install n8n Skills

Skills give Claude expert n8n knowledge. They are installed once per machine to
`~/.claude/skills/` and available in all future sessions automatically.

**Windows (PowerShell)**
```powershell
# Clone the skills repo
git clone --depth=1 https://github.com/czlonkowski/n8n-skills "$env:TEMP\n8n-skills"

# Create the skills directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills" | Out-Null

# Copy all 7 skills
Copy-Item -Recurse "$env:TEMP\n8n-skills\skills\*" "$env:USERPROFILE\.claude\skills\"

# Clean up
Remove-Item -Recurse -Force "$env:TEMP\n8n-skills"

# Verify
Get-ChildItem "$env:USERPROFILE\.claude\skills"
```

**macOS / Linux**
```bash
# Clone the skills repo
git clone --depth=1 https://github.com/czlonkowski/n8n-skills /tmp/n8n-skills

# Create the skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy all 7 skills
cp -r /tmp/n8n-skills/skills/* ~/.claude/skills/

# Clean up
rm -rf /tmp/n8n-skills

# Verify
ls ~/.claude/skills
```

**Expected output after verify:**
```
n8n-code-javascript
n8n-code-python
n8n-expression-syntax
n8n-mcp-tools-expert
n8n-node-configuration
n8n-validation-expert
n8n-workflow-patterns
```

---

### Step 5: Start a Claude Session

1. In VS Code, click the **Claude Code** icon in the Activity Bar (left sidebar)
2. Start a new conversation

Claude Code will automatically:
- Read `CLAUDE.md` and load the workflow architect instructions
- Connect to your n8n instance using `.mcp.json`
- Load all 7 skills from `~/.claude/skills/`

---

### Step 6: Verify the Connection

In the Claude Code chat, type:

```
Run n8n_health_check
```

A successful response looks like:

```
✓ Connected to n8n instance
  URL: https://your-instance.app.n8n.cloud
  Version: 1.x.x
  API: reachable
```

If it fails, see the Troubleshooting section below.

---

## Building Workflows

Just describe what you want. Claude handles the rest.

**Example 1 — Webhook to Slack alert**
```
Build a workflow that receives a webhook from Stripe, checks if the payment
amount exceeds $500, and sends a Slack alert with the customer name and amount.
```

**Example 2 — Scheduled data sync**
```
Create a workflow that runs every day at 8am, fetches new rows added in the
last 24 hours from a Google Sheet, and inserts them into an Airtable base.
```

**Example 3 — Improve an existing workflow**
```
Review my workflow called "Process orders" and add proper error handling
so failures send me an email notification.
```

Claude will ask clarifying questions, show you the plan, build it, validate it, test it,
and add documentation — all before declaring the workflow done.

---

## Available MCP Tools

### Core Tools — No API Key Required

| Tool | What It Does |
|---|---|
| `tools_documentation` | Get usage guidance for any MCP tool |
| `search_nodes` | Search 1,084+ n8n nodes by name or function |
| `get_node` | Get node documentation, properties, and version info |
| `validate_node` | Validate a single node's configuration |
| `validate_workflow` | Validate a complete workflow JSON |
| `search_templates` | Search 2,709 community templates |
| `get_template` | Retrieve a template's full workflow JSON |

### Management Tools — Require n8n API Key

| Tool | What It Does |
|---|---|
| `n8n_health_check` | Verify API connectivity and instance status |
| `n8n_list_workflows` | List all workflows in your instance |
| `n8n_get_workflow` | Get a workflow's full details by ID |
| `n8n_create_workflow` | Create a new workflow (starts inactive) |
| `n8n_update_full_workflow` | Replace a workflow entirely |
| `n8n_update_partial_workflow` | Make targeted incremental changes |
| `n8n_delete_workflow` | Delete a workflow permanently |
| `n8n_validate_workflow` | Validate a deployed workflow by ID |
| `n8n_autofix_workflow` | Auto-fix common validation errors |
| `n8n_workflow_versions` | View history and roll back a workflow |
| `n8n_deploy_template` | Deploy a template from n8n.io with auto-fix |
| `n8n_test_workflow` | Trigger a test execution |
| `n8n_executions` | List and inspect execution records |

---

## n8n Skills Reference

Skills activate automatically — no commands needed.

| Skill | Activates When |
|---|---|
| `n8n-expression-syntax` | Writing `{{ }}` expressions, using `$json`, `$node`, debugging expression errors |
| `n8n-mcp-tools-expert` | Searching nodes, validating configs, managing workflows via MCP |
| `n8n-workflow-patterns` | Designing workflows, choosing architecture, connecting nodes |
| `n8n-validation-expert` | Debugging validation errors, fixing broken workflows |
| `n8n-node-configuration` | Configuring specific nodes, understanding property dependencies |
| `n8n-code-javascript` | Writing Code node JavaScript, accessing `$input`, `$json`, `$node` |
| `n8n-code-python` | Writing Code node Python, understanding Python limitations in n8n |

---

## Troubleshooting

### MCP server won't connect

1. Check that `N8N_API_URL` in `.mcp.json` has no trailing slash
   - Correct: `https://yourname.app.n8n.cloud`
   - Wrong: `https://yourname.app.n8n.cloud/`
2. Confirm your API key is still valid — open n8n → Settings → API and check it hasn't been deleted
3. For self-hosted instances, make sure n8n is running before opening VS Code
4. Restart VS Code after editing `.mcp.json`

### Skills not activating

1. Confirm all 7 folders exist in `~/.claude/skills/` (run the verify command from Step 4)
2. Each skill folder must contain a `skill.md` file — if folders are empty, re-run the install
3. Restart VS Code after installing skills

### `npx n8n-mcp` is slow to start

The first run downloads the package (~30 seconds). Subsequent runs use the npx cache and
start in under 2 seconds. This is normal.

### Workflow was created but I can't see it in n8n

All workflows created by Claude start **inactive** by design. In n8n, go to your workflows
list — inactive workflows appear with a grey toggle. Activate it manually once you've
reviewed it, or ask Claude to activate it for you.

### n8n API Key expired or missing

n8n Cloud API keys do not expire by default, but they can be deleted. If the health check
fails with an authentication error:
1. Go to n8n → Settings → API
2. Create a new API key
3. Update `N8N_API_KEY` in `.mcp.json`
4. Restart VS Code

---

## References

- [n8n-mcp server](https://github.com/czlonkowski/n8n-mcp) — MCP tools for n8n by @czlonkowski
- [n8n-skills](https://github.com/czlonkowski/n8n-skills) — Claude Code skills for n8n by @czlonkowski
- [n8n Documentation](https://docs.n8n.io) — Official n8n docs
- [n8n Template Library](https://n8n.io/workflows/) — 2,709 community workflow templates
- [Claude Code Extension](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code) — VS Code Marketplace
