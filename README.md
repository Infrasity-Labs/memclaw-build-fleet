# MemClaw + OpenClaw Fleet Complete Setup Guide

> **A 5-agent AI fleet with shared memory, built from scratch on Windows.**
> Each agent recalls what the previous one wrote before acting. Memory compounds.

![Fleet Status](https://img.shields.io/badge/Fleet-5%20Agents%20Live-brightgreen)
![MemClaw](https://img.shields.io/badge/MemClaw-v0.9.31-orange)
![OpenClaw](https://img.shields.io/badge/OpenClaw-v2026.4.29-blue)
![Model](https://img.shields.io/badge/Model-Claude%20Haiku%204.5%20via%20AIsa-purple)

---

## What This Repo Contains

```
├── index.html          # SaaS landing page built by the 4-skill agent pipeline
└── README.md           # Complete setup guide + tested context
```

The `index.html` was produced by a 4-agent pipeline where each agent recalled
the previous agent's MemClaw memories before building on them:

```
Frontend Agent → Performance Agent → SEO Agent → Code Review Agent
     ↓                  ↓                ↓               ↓
 memclaw_write     memclaw_recall    memclaw_recall  memclaw_recall
                   + memclaw_write   + memclaw_write  + memclaw_write
                                                      + memclaw_insights
```

All 6 memories written to MemClaw fleet are visible at `memclaw.net/prism`.

---

## Architecture

```
┌─────────────────────────────────────────────┐
│           OpenClaw Gateway (local)           │
│                                              │
│  master (orchestrator) ← highest priority   │
│  ├── frontend    (Claude Haiku 4.5 )  │
│  ├── performance (Claude Haiku 4.5 )  │
│  ├── seo         (Claude Haiku 4.5 )  │
│  └── codereview  (Claude Haiku 4.5 )  │
│                                              │
│  All agents share one MemClaw fleet memory   │
└─────────────────────────────────────────────┘
         ↕ memclaw_write / memclaw_recall
┌─────────────────────────────────────────────┐
│        MemClaw Managed Platform              │
│        memclaw.net · tenant: sanyog          │
│        fleet: webpage-fleet                  │
│        6 memories · 4 agents · 0 stale      │
└─────────────────────────────────────────────┘

```

---

## Prerequisites

- Windows 10/11
- Node.js v20+ (we used v24.15.0)
- Git for Windows (includes bash)
- An account at [memclaw.net](https://memclaw.net) (free)


---

## Step-by-Step Setup

### Step 1 : Install OpenClaw

```powershell
npm install -g openclaw@latest
openclaw --version
# OpenClaw 2026.4.29 (a448042)
```

### Step 2 : Run Onboarding

```powershell
openclaw onboard --install-daemon
```

During onboarding:
- Skip channel selection for now
- Skip skills configuration
- Skip hooks
- Select "Hatch in Terminal" to start the TUI

### Step 3 : Configure a Model Provider
![Configure  a Model Provider](https://infrasity-pull-zone.b-cdn.net/memclaw/step3.jpeg)





### Step 4 : Install MemClaw Plugin

Get your MemClaw API key from [memclaw.net](https://memclaw.net) (free account).

Run in Git Bash:
```bash
# Save the install payload
[System.IO.File]::WriteAllText("payload.json", '{"fleet_id":"fleet","api_url":"https://memclaw.net","api_key":"YOUR_MC_API_KEY"}')

# Fetch and run the installer
curl -ks -X POST "https://memclaw.net/api/v1/install-plugin" \
  -H "Content-Type: application/json" \
  --data-binary "@payload.json" > /tmp/memclaw-install.sh

# Fix Windows hostname compatibility
sed -i 's/hostname -s/hostname/g' /tmp/memclaw-install.sh
bash /tmp/memclaw-install.sh
```

Expected output:
```
=== Installation complete ===
Plugin directory: ~/.openclaw/plugins/memclaw
API URL: https://memclaw.net
Fleet ID: fleet
```

### Step 5 : Configure the 5-Agent Fleet

![Configure the 5-Agent Fleet](https://infrasity-pull-zone.b-cdn.net/memclaw/step5.png)

Update `~/.openclaw/openclaw.json` to add the fleet. The key section:

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "claude-haiku-4-5-20251001" },
      "workspace": "workspace"
    },
    "list": [
      { "id": "master",      "name": "Master Orchestrator", "model": "claude-haiku-4-5-20251001", "workspace": "workspace-master",      "default": true },
      { "id": "frontend",    "name": "Frontend Agent",      "model": "claude-haiku-4-5-20251001", "workspace": "workspace-frontend"          },
      { "id": "performance", "name": "Performance Agent",   "model": "claude-haiku-4-5-20251001", "workspace": "workspace-performance"       },
      { "id": "seo",         "name": "SEO Agent",           "model": "claude-haiku-4-5-20251001", "workspace": "workspace-seo"               },
      { "id": "codereview",  "name": "Code Review Agent",   "model": "claude-haiku-4-5-20251001", "workspace": "workspace-codereview"        }
    ]
  },
  "plugins": {
    "entries": {
      "memclaw": { "enabled": true, "config": {} }
    },
    "slots": { "memory": "memclaw" },
    "load": { "paths": ["C:/Users/YOUR_NAME/.openclaw/plugins/memclaw"] }
  },
  "tools": {
    "alsoAllow": [
      "memclaw_recall","memclaw_write","memclaw_manage","memclaw_doc",
      "memclaw_list","memclaw_entity_get","memclaw_tune","memclaw_insights","memclaw_evolve"
    ]
  }
}
```

> **Windows gotcha**: OpenClaw has an auto-restore feature that restores the config
> from `openclaw.json.last-good` if the new file is smaller. Override ALL backup files:

```powershell
$content = Get-Content "$env:USERPROFILE\Downloads\openclaw.json" -Raw
[System.IO.File]::WriteAllText("$env:USERPROFILE\.openclaw\openclaw.json", $content)
[System.IO.File]::WriteAllText("$env:USERPROFILE\.openclaw\openclaw.json.bak", $content)
[System.IO.File]::WriteAllText("$env:USERPROFILE\.openclaw\openclaw.json.last-good", $content)
[System.IO.File]::WriteAllText("$env:USERPROFILE\.openclaw\openclaw.json.good-backup", $content)
```

### Step 6 : Install MemClaw Skill on All Agents

```powershell
openclaw skills install memclaw
openclaw skills install memclaw --agent frontend
openclaw skills install memclaw --agent performance
openclaw skills install memclaw --agent seo
openclaw skills install memclaw --agent codereview
```

### Step 7 : Restart Gateway and Verify

```powershell
openclaw gateway stop
openclaw gateway start
openclaw agents list
```

Expected output:
```
Agents:
- master (default) (Master Orchestrator)
  Model: claude-haiku-4-5-20251001
- frontend (Frontend Agent)
  Model: claude-haiku-4-5-20251001
- performance (Performance Agent)
  Model: claude-haiku-4-5-20251001
- seo (SEO Agent)
  Model: claude-haiku-4-5-20251001
- codereview (Code Review Agent)
  Model: claude-haiku-4-5-20251001
```

### Step 8 :  Open the Web UI
![Web UI of openClaw to chat](https://infrasity-pull-zone.b-cdn.net/memclaw/step8.jpeg)

```powershell
openclaw dashboard
```

This opens `http://127.0.0.1:18789` in your browser with the token pre-filled.

---

## Running the 4-Skill Pipeline

In the OpenClaw web UI chat, type:

```
Now act as each of the 5 agents in sequence.
For each agent use memclaw_write to save real memories with tenant_id "sanyog":

1. As frontend-agent: write a memory about building a SaaS landing page
   with semantic HTML5, CSS Grid, critical CSS
2. As performance-agent: use memclaw_recall to recall frontend memories,
   then write performance audit findings
3. As seo-agent: use memclaw_recall to recall all memories,
   then write SEO recommendations
4. As codereview-agent: use memclaw_recall to recall everything,
   write final merge verdict

After all 4, use memclaw_insights to analyze the fleet memory.
```

Watch the MemClaw dashboard at `memclaw.net/prism` — memory counter goes up live.

---

## Tested Context : What Actually Worked
![All skills using shared memory](https://infrasity-pull-zone.b-cdn.net/memclaw/testedcontext.png)
### ✅ What Worked

| Component | Status | Notes |
|---|---|---|
| OpenClaw install via npm | ✅ | Use `openclaw@latest`, add to PATH |

| MemClaw plugin install | ✅ | Run in Git Bash, patch `hostname -s` → `hostname` |
| 5-agent fleet config | ✅ | Must overwrite ALL backup files to prevent auto-restore |
| MemClaw skill per agent | ✅ | `openclaw skills install memclaw --agent <id>` |
| Real `memclaw_write` tool call | ✅ | Verified with Memory ID in response |
| MemClaw Prism dashboard | ✅ | 6 memories visible across 4 agents |


### ⚠️ Issues Encountered and Fixes

**Issue 1: `openclaw` not found after npm install**
```
Fix: Add npm global bin to PATH
$env:PATH += ";C:\Users\YOUR_NAME\AppData\Roaming\npm"
```

**Issue 2: Gemini API 503 overload**
```
Fix: Switch to some other model 
Run the model setup script from their GitHub
```

**Issue 3: MemClaw install fails : `hostname -s` not found**
```
Fix: Save the script, patch it, then run:
sed -i 's/hostname -s/hostname/g' /tmp/memclaw-install.sh
bash /tmp/memclaw-install.sh
```

**Issue 4: OpenClaw auto-restores old config**
```
Fix: Overwrite openclaw.json.last-good as well as openclaw.json
The auto-restore compares file sizes — write new config to all backup paths
```

**Issue 5: Workspace paths rejected by MemClaw plugin**
```
Fix: Use relative paths in agents.list config
"workspace": "workspace-master" not "C:\\Users\\...\\workspace-master"
```

**Issue 6: Agent session stuck / no response**
```
Fix: Clear sessions and restart
Remove-Item "$env:USERPROFILE\.openclaw\agents\main\sessions" -Recurse -Force
openclaw gateway stop && openclaw gateway start
```

**Issue 7: gemma3:4b doesn't support tools**
```
Fix: Use Claude Haiku instead — full tool support
Or set "supportsTools": true in the model config (may not work for all models)
```

**Issue 8: `memclaw_write` writes to wrong tenant**
```
Fix: Explicitly pass tenant_id in every tool call
Use your MemClaw dashboard tenant name (e.g., "sanyog" not "webpage-fleet")
```

### 📊 Fleet Memory Results

After running the pipeline, MemClaw Prism shows:

```
TOTAL: 6 memories
Types: decision (5), fact (1)
Agents: frontend-agent (2), performance-agent (1), seo-agent (2), code-review-agent (1)

Memory weights (importance scores):
- frontend build plan:     0.95
- performance CWV audit:   0.90
- SEO title/meta audit:    0.90
- SEO schema/OG audit:     0.88
- Code review verdict:     0.95 (APPROVED)
- Frontend fact:           0.70
```

---

## Key Config Files

### `~/.openclaw/plugins/memclaw/.env`
```env
MEMCLAW_API_URL=https://memclaw.net
MEMCLAW_API_KEY=mc_your_key_here
MEMCLAW_FLEET_ID=fleet
MEMCLAW_TENANT_ID=your_tenant_id
MEMCLAW_NODE_NAME=your_hostname
```

### MemClaw MCP Config (for Claude Desktop / Claude Code)
```json
{
  "mcpServers": {
    "memclaw": {
      "url": "https://memclaw.net/mcp",
      "headers": { "X-API-Key": "mc_your_api_key_here" }
    }
  }
}
```

---

## 9 MemClaw Tools Available to All Agents

| Tool | Purpose |
|---|---|
| `memclaw_write` | Write memories to fleet (single or batch up to 100) |
| `memclaw_recall` | Hybrid semantic + keyword recall with graph retrieval |
| `memclaw_manage` | Read, update, transition, delete individual memories |
| `memclaw_list` | Filter by type/status/agent, cursor-paginate |
| `memclaw_doc` | Structured document CRUD on named JSON collections |
| `memclaw_entity_get` | Look up knowledge graph entity by UUID |
| `memclaw_tune` | Tune per-agent retrieval params (top_k, min_similarity) |
| `memclaw_insights` | Analyze memory store — contradictions, patterns, stale |
| `memclaw_evolve` | Report outcomes → adjusts weights (Karpathy Loop) |

---

## Resources

- [MemClaw GitHub](https://github.com/caura-ai/caura-memclaw)
- [MemClaw Managed Platform](https://memclaw.net)
- [OpenClaw Docs](https://docs.openclaw.ai)


---

## License

The setup guide and website in this repo are MIT licensed.
MemClaw itself is Apache 2.0 licensed.
