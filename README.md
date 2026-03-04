# n8n Workflow Builder with Claude Code

A project workspace for building production-ready n8n workflows using Claude Code as an AI
workflow architect. Claude connects directly to your n8n instance via the n8n-mcp server and
uses domain-specific skills to design, build, validate, and test workflows.

---

## What's Included

| File | Purpose |
|---|---|
| `CLAUDE.md` | Persistent instructions that define Claude's role, tools, quality standards, and workflow creation process |
| `.mcp.json` | MCP server configuration — connects Claude Code to your n8n instance |

---

## Prerequisites

- [Node.js](https://nodejs.org/) v18 or later (includes `npx`)
- Claude Code Extension in VS Code
- An n8n instance (self-hosted or [n8n Cloud](https://app.n8n.cloud))
- An n8n API key (see below)

---

## Setup

### 0. Create a Folder

### 1. Clone the Repository

```bash
cd <Your Folder Path>
git clone https://github.com/bridgenetlab/n8nClaude.git
```

### 2. Get Your n8n API Key

1. Open your n8n instance
2. Go to **Settings → API**
3. Click **Create API Key**
4. Copy the generated key

### 3. Configure `.mcp.json`

Edit `.mcp.json` and replace the placeholder values with your own:

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

> The `N8N_API_URL` is the root URL of your n8n instance (e.g. `http://localhost:5678` for
> self-hosted or `https://yourname.app.n8n.cloud` for n8n Cloud).

### 4. Install n8n-mcp
```bash
npx n8n-mcp
```

### 4. Install n8n Skills

Skills inject expert n8n knowledge into Claude's context automatically. Run once per machine:

```bash
# Clone and copy skills to Claude's skills directory
git clone --depth=1 https://github.com/czlonkowski/n8n-skills /tmp/n8n-skills

# macOS / Linux
cp -r /tmp/n8n-skills/skills/* ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse /tmp/n8n-skills/skills/* $env:USERPROFILE\.claude\skills\

rm -rf /tmp/n8n-skills
```

This installs the following 7 skills to `~/.claude/skills/`:

| Skill | Activates When |
|---|---|
| `n8n-expression-syntax` | Writing `{{ }}` expressions, accessing `$json`, `$node` |
| `n8n-mcp-tools-expert` | Searching for nodes, validating configurations, managing workflows |
| `n8n-workflow-patterns` | Designing workflows, choosing architecture |
| `n8n-validation-expert` | Debugging validation errors, fixing broken workflows |
| `n8n-node-configuration` | Configuring specific nodes, understanding property dependencies |
| `n8n-code-javascript` | Writing Code node JavaScript |
| `n8n-code-python` | Writing Code node Python |

Skills activate automatically — no commands needed.

### 5. Open the Project in Claude Code (Visual Studio Code)

Claude Code will automatically:
- Read `CLAUDE.md` and load the workflow architect instructions
- Load `.mcp.json` and start the n8n-mcp server
- Make all 20 n8n MCP tools available in the session

---

## How It Works

```
You describe a workflow need
        ↓
Claude asks clarifying questions (trigger, data, error handling, credentials)
        ↓
Claude checks existing workflows and searches templates
        ↓
Claude plans the workflow and gets your approval
        ↓
Claude builds it directly in your n8n instance via MCP
        ↓
Claude validates, tests, and documents the workflow
```

### Available MCP Tools

Claude has access to 20 tools from the [n8n-mcp](https://github.com/czlonkowski/n8n-mcp) server:

**Core tools (no API key required)**

| Tool | Purpose |
|---|---|
| `tools_documentation` | Self-help for any MCP tool |
| `search_nodes` | Search 1,084+ n8n nodes |
| `get_node` | Get node documentation and properties |
| `validate_node` | Validate a single node configuration |
| `validate_workflow` | Validate a full workflow JSON |
| `search_templates` | Search 2,709 community templates |
| `get_template` | Retrieve a template's workflow JSON |

**Management tools (require API key)**

| Tool | Purpose |
|---|---|
| `n8n_health_check` | Verify API connectivity |
| `n8n_list_workflows` | List all workflows |
| `n8n_get_workflow` | Get workflow details by ID |
| `n8n_create_workflow` | Create a new workflow |
| `n8n_update_full_workflow` | Replace a workflow entirely |
| `n8n_update_partial_workflow` | Make targeted incremental changes |
| `n8n_delete_workflow` | Delete a workflow |
| `n8n_validate_workflow` | Validate a deployed workflow |
| `n8n_autofix_workflow` | Auto-fix common validation errors |
| `n8n_workflow_versions` | View history and roll back |
| `n8n_deploy_template` | Deploy a template from n8n.io |
| `n8n_test_workflow` | Trigger a test execution |
| `n8n_executions` | Inspect execution records |

---

## Example Prompts

```
Build a workflow that receives a webhook from Stripe, checks if the payment amount
exceeds $500, and sends a Slack alert if it does.
```

```
Create a scheduled workflow that runs every Monday at 9am, fetches new rows from
a Google Sheet, and sends a summary email via Gmail.
```

```
I have a workflow called "Sync contacts" — review it and add proper error handling.
```

---

## Troubleshooting

**MCP server not connecting**
- Confirm `N8N_API_URL` in `.mcp.json` is the correct root URL (no trailing slash)
- Confirm the API key is valid and hasn't expired
- Run `n8n_health_check` inside a Claude Code session to diagnose

**Skills not activating**
- Verify skill folders are in `LocalDisk/Users/<Your User>/.claude/skills/` (each folder should contain a `skill.md`)
- Restart Claude Code after installation

**`npx n8n-mcp` slow to start**
- First run downloads the package — subsequent runs use the npx cache and are faster

---

## References

- [n8n-mcp server](https://github.com/czlonkowski/n8n-mcp) by @czlonkowski
- [n8n-skills](https://github.com/czlonkowski/n8n-skills) by @czlonkowski
- [n8n Documentation](https://docs.n8n.io)
- [Claude Code Documentation](https://claude.ai/code)
