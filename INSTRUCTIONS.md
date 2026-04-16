# n8n AI Workflow Builder — Agent Instructions

> **This is the canonical instruction file.** All agent-specific files (CLAUDE.md, AGENTS.md,
> .cursorrules, .windsurfrules, .github/copilot-instructions.md) are auto-generated copies.
> Edit this file, then run `npm run sync:instructions` to update all copies.

> **This file teaches any AI coding agent how to build, manage, and deploy n8n workflows.**
> It works with Claude Code, Codex, Cursor, Windsurf, Copilot, and any agent that reads project instructions.

## Project Purpose

This project enables AI-powered n8n workflow creation through natural language. You (the AI agent) have tools and knowledge to build, validate, and manage n8n workflows on the user's n8n instance.

## First-Time Setup

If the user hasn't set up the project yet, read [SETUP.md](SETUP.md) and guide them through it conversationally.

---

## Available Tools

### CLI Scripts

| Command | Description |
|---------|-------------|
| `npm run pull` | Download all workflows from n8n instance |
| `npm run push workflows/file.json` | Upload/update a workflow to n8n |
| `npm run validate workflows/file.json` | Validate a single workflow |
| `npm run validate:all` | Validate all workflows |
| `npm run run -- <id> --input data.json` | Run a workflow via webhook trigger |
| `npm run execute -- <id> --wait` | Execute a workflow and poll for results |
| `npm run sync:pull` | Pull + validate all |
| `npm run sync:push` | Validate + push changed workflows |

### Skills

If n8n skills are installed in your agent's skills directory, you have access to 8 expert n8n skills:

| Skill | Purpose | Priority |
|-------|---------|----------|
| **n8n-mcp-tools-expert** | MCP tool usage patterns | Highest - use first |
| **n8n-expression-syntax** | Expression syntax (`{{}}`) | When working with data transformations |
| **n8n-workflow-patterns** | Architecture & patterns | When designing workflows |
| **n8n-validation-expert** | Fix validation errors | When workflows fail validation |
| **n8n-node-configuration** | Node setup | When configuring specific nodes |
| **n8n-code-javascript** | JavaScript in Code nodes | For Code node JS |
| **n8n-code-python** | Python in Code nodes | For Code node Python |
| **n8n-workflow-auditor** | Design quality audit | When reviewing workflows |

Skills are in the `skills/n8n/` directory. See [skills/n8n/README.md](skills/n8n/README.md) for install instructions for your specific agent.

### MCP Server

If the n8n-mcp server is configured, you have direct access to:
- **1,084 n8n nodes** (537 core + 547 community) with documentation
- **2,709 workflow templates** with metadata
- Real-world examples and patterns

**MCP Tools Available:**

| Tool | Purpose | When to Use |
|------|---------|-------------|
| `search_nodes` | Find nodes by functionality | Discovering which node to use |
| `get_node` | Detailed node documentation | Configuring a specific node — get exact parameters |
| `search_templates` | Browse workflow templates | Finding similar workflows to learn patterns |
| `get_template` | Retrieve specific template | Adapting a proven template |
| `validate_workflow` | **Validate workflow JSON against n8n schema** | **MANDATORY before every push** — catches bad typeVersions, invalid parameters, broken connections, expression errors |
| `n8n_autofix_workflow` | **Auto-fix common validation errors** | After validation fails — fixes typeVersion mismatches, expression format, connection issues |
| `n8n_validate_workflow` | Validate an already-pushed workflow by ID | Post-push verification or auditing existing workflows |
| `n8n_create_workflow` | Create workflow directly via API | Alternative to `npm run push` — API enforces schema |
| `n8n_generate_workflow` | Generate workflow from natural language | When n8n's own engine can produce a better starting point than hand-crafting JSON |

See `.mcp.json.example` for the configuration template.

---

## How to Create a Workflow

### MANDATORY Process — Follow Every Time

**Why this process exists:** Workflows built from an agent's training data or incomplete docs often break on import — wrong typeVersions, invalid parameters, broken canvas. The fix is to **validate against the real n8n instance before pushing**, using MCP tools.

When the user asks you to create or modify a workflow, follow this process **without skipping steps**:

### 1. Understand Requirements

Ask clarifying questions if needed:
- What trigger should start the workflow?
- What data sources/destinations?
- What transformations are needed?
- Any specific n8n nodes to use?

### 2. Research (Critical — do not guess)

**If you have MCP access:**
1. Search for nodes: `search_nodes` — get the exact node type name, not from memory
2. Get node documentation: `get_node` — read FULL parameter schema for every node you'll use. Note the correct typeVersion, required fields, valid option values.
3. Search for templates: `search_templates` — find similar workflows to copy proven patterns
4. Pull existing working workflows: `npm run pull` — check typeVersions and parameter structures from workflows already running on the user's instance

**If you don't have MCP access:**
- Use the knowledge in this file and the skills directory
- Pull existing workflows from the user's instance to learn patterns: `npm run pull`
- Reference n8n docs at https://docs.n8n.io

### 3. Design the Workflow Structure

Plan before building:
1. Trigger node (how it starts)
2. Processing nodes (what it does)
3. Output nodes (where results go)
4. Error handling (what happens when things fail)

### 4. Create the Workflow JSON

Build the workflow file in `workflows/`:
1. Create JSON structure with all nodes
2. Configure node parameters using EXACT structures from step 2 — do NOT invent parameter names or option values. Copy from `get_node` docs or working workflows.
3. Set up connections between nodes
4. Add error handling
5. Configure workflow settings

**For new workflows, do NOT include an `"id"` field** — the push script will assign one.

### 5. Local Validation

```bash
npm run validate workflows/new-workflow.json
```

Fix any structural errors before proceeding.

### 6. MCP Validation — MANDATORY (if MCP available)

Use the `validate_workflow` MCP tool with the full workflow JSON:
- Profile: `"strict"`
- validateNodes: `true`
- validateConnections: `true`
- validateExpressions: `true`

This validates against n8n's actual node schemas — catches:
- Wrong typeVersions
- Invalid parameter values/options
- Broken connections
- Expression syntax errors
- Missing required fields

**DO NOT SKIP THIS STEP.** It is the primary defense against "works in your agent, breaks in n8n" failures.

### 7. Auto-fix if Validation Fails

If step 6 returns errors:
1. Review each error — understand what's wrong
2. Use `n8n_autofix_workflow` if the workflow is already pushed, or manually fix the JSON based on error messages
3. Re-validate until clean

Common auto-fixable issues:
- typeVersion mismatches (auto-corrected to instance version)
- Expression format errors (auto-corrected)
- Connection structure issues (auto-corrected)

### 8. Push to n8n

```bash
npm run push workflows/new-workflow.json
```

Alternative: use `n8n_create_workflow` MCP tool for direct API creation with schema enforcement.

The push script will:
- Create the workflow on n8n (POST)
- Write the assigned ID back into the JSON file
- Rename the file to include the workflow ID

### 9. Post-Push Verification

After push, verify the workflow loaded correctly:
- Use `n8n_validate_workflow` (by ID) to confirm instance-side validity
- Ask the user to check the canvas if this is a complex workflow

### 10. Commit

```bash
git add workflows/
git commit -m "Add workflow: description"
```

---

## Workflow JSON Structure

```json
{
  "name": "Descriptive Workflow Name",
  "nodes": [
    {
      "parameters": {},
      "id": "unique-uuid",
      "name": "Node Name",
      "type": "n8n-nodes-base.nodeName",
      "typeVersion": 1,
      "position": [x, y]
    }
  ],
  "connections": {
    "Node Name": {
      "main": [[{"node": "Next Node", "type": "main", "index": 0}]]
    }
  },
  "settings": {
    "executionOrder": "v1"
  }
}
```

---

## Critical Knowledge

### typeVersions — ALWAYS Verify Dynamically

**DO NOT rely on a static table.** n8n updates change typeVersions. Always verify at workflow creation time using these methods (in priority order):

#### Method 1: MCP Validation (preferred)
Use `validate_workflow` with `strict` profile — it will flag any wrong typeVersion and tell you the correct one. Then use `n8n_autofix_workflow` to auto-correct.

#### Method 2: Check existing workflows on the instance
```bash
npm run pull
# Then grep for the node type to see what version is actually in use:
grep -A2 '"type": "n8n-nodes-base.httpRequest"' workflows/*.json | grep typeVersion
```

#### Method 3: MCP node documentation
Use `get_node` to check the node's available versions.

**Why not a static table?** A hardcoded table goes stale the moment n8n updates. The three methods above always give you the truth from the actual instance.

### Read-Only Fields (Never Include When Pushing)

These fields cause API push errors:
- `updatedAt`, `createdAt`
- `active` (read-only on PUT — `n8n-manager.js` calls `POST /workflows/{id}/deactivate` before PUT if the workflow is active, then warns you to re-activate from UI)
- `isArchived`
- `versionId`, `activeVersionId`, `versionCounter`
- `triggerCount`
- `shared`, `tags`
- `activeVersion`, `staticData`, `meta`, `pinData`

The `n8n-manager.js` push script strips these automatically. If crafting raw API calls manually, omit them.

### SplitInBatches (Loop Over Items) — Output Indices

**Output 0 = "done", Output 1 = "loop" — the opposite of what you'd expect.**

| Output index | Label | Purpose |
|---|---|---|
| 0 | done | Fires when ALL batches are finished |
| 1 | loop | Sends current batch items each iteration |

**Correct connection pattern:**
```json
"Loop Over Items": {
  "main": [
    [],
    [{ "node": "First Node In Loop", "type": "main", "index": 0 }]
  ]
}
```
The last node in the loop connects back to the SplitInBatches input:
```json
"Last Node In Loop": {
  "main": [[{ "node": "Loop Over Items", "type": "main", "index": 0 }]]
}
```

### Retry Loops (Backward Connections)

n8n supports backward connections (cycles). Use them for retry patterns:

1. A **Check node** (Code) validates output and tracks a retry counter
2. An **IF node** gates on the check result
3. On failure: a **Fix/Retry chain** connects back to the Check node
4. The Check node enforces a max retry limit to prevent infinite loops

```js
// Check node example
const structRetryCount = $json.structRetryCount || 0;
const MAX_RETRIES = 3;
return { ...$json, structOk: isValid || structRetryCount >= MAX_RETRIES, structRetryCount };
```

### Expression Syntax

Use `{{ }}` for dynamic content in node parameter fields:

```
{{ $json.field }}                    - Current node data
{{ $json.body.name }}               - Webhook data (always under .body!)
{{ $node["Node Name"].json.field }} - Data from specific node
{{ $now.toFormat('yyyy-MM-dd') }}   - Current date
```

**Critical:** Webhook data is under `$json.body`, NOT `$json` directly.

**In Code nodes:** Use JavaScript directly, NOT expressions:
```javascript
// Code node — NO {{ }}
const email = $input.first().json.email;
```

### IF Node Configuration (typeVersion 2.2+)

```json
{
  "parameters": {
    "conditions": {
      "options": {
        "caseSensitive": true,
        "leftValue": "",
        "typeValidation": "loose"
      },
      "conditions": [
        {
          "id": "condition-1",
          "leftValue": "={{ $json.field }}",
          "rightValue": "value",
          "operator": {
            "type": "string",
            "operation": "equals"
          }
        }
      ],
      "combinator": "and"
    },
    "options": {}
  },
  "type": "n8n-nodes-base.if",
  "typeVersion": 2.3
}
```

### Common Import Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Could not find property option" | Wrong typeVersion | Use `validate_workflow` (MCP) to detect, `n8n_autofix_workflow` to fix |
| Broken/missing node on canvas | Sub-node used standalone | Use HTTP Request node instead |
| API push rejected (400) | Read-only fields in JSON | The push script handles this, but avoid them in manual calls |
| Blank canvas after push | Structure mismatches | Try manual import: n8n UI > Menu > Import from File |

### Sub-Nodes (Cannot Be Used Standalone)

These nodes only work inside AI Agent workflows, NOT as standalone nodes:
- `n8n-nodes-langchain.embeddingsOpenAi`
- `n8n-nodes-langchain.lmChatOpenAi`
- Other langchain sub-nodes

**Use HTTP Request node to call APIs directly instead.**

---

## Workflow Naming Convention

```
<descriptive_name>_<workflow_id>.json
```

- Use lowercase with underscores
- Be descriptive: `gmail_daily_digest_abc123.json` not `workflow1.json`
- The push script adds the workflow ID automatically after first push

---

## Error Handling Best Practices

Every workflow should include error handling:

1. **Error Trigger node** — catches workflow-level errors
2. **IF nodes** — for conditional logic and error checking
3. **Continue On Fail** — per-node setting for graceful degradation
4. **Stop And Error nodes** — for critical failures that should halt execution

---

## Code Node Best Practices

### JavaScript (Preferred)

```javascript
// Always return array of {json: {...}} objects
const items = $input.all();
return items.map(item => ({
  json: {
    ...item.json,
    processed: true
  }
}));
```

Key rules:
- Must return `[{json: {...}}]` format
- Use `$input.all()` for all items, `$input.first()` for single item
- Webhook data is under `.body`: `$json.body.email`
- `$helpers.httpRequest()` for HTTP calls inside Code nodes
- `DateTime` (Luxon) for date/time operations

### Python

```python
items = _input.all()
return [{"json": {**item["json"], "processed": True}} for item in items]
```

Key rules:
- Same return format: `[{"json": {...}}]`
- Use `_input.all()`, `_input.first()`, `_input.item`
- **No external libraries** — standard library only (json, datetime, re, etc.)
- Use `.get()` for safe dictionary access

---

## Security

### Never Commit
- `.env` file (contains API key)
- `.mcp.json` file (contains MCP config)
- API keys or credentials in workflow JSON

### Always Check
- `.gitignore` includes `.env` and `.mcp.json`
- Workflows don't contain hardcoded credentials
- Credentials use n8n's credential system, not inline values

---

## Quality Checklist

Before considering a workflow complete:

- [ ] Nodes researched using MCP (`search_nodes` + `get_node` for each node type)
- [ ] Similar templates reviewed (`search_templates`)
- [ ] typeVersions verified against instance (grep existing workflows or MCP validation)
- [ ] Node parameters copied from docs/working workflows, NOT guessed
- [ ] Connections are correct
- [ ] Error handling included
- [ ] Local validation passes (`npm run validate`)
- [ ] **MCP validation passes (`validate_workflow` with `strict` profile)** — most important step
- [ ] Auto-fix applied if MCP validation found issues (`n8n_autofix_workflow`)
- [ ] Pushed to n8n (`npm run push`)
- [ ] Post-push verification done (`n8n_validate_workflow` by ID)
- [ ] File renamed with workflow ID
- [ ] Committed to git

---

## Common Pitfalls

### Don't
- **EVER skip MCP validation (`validate_workflow`)** — the #1 cause of broken workflows
- **Guess node parameters from training data** — always use `get_node` MCP docs or copy from working workflows
- **Rely on a static typeVersion table** — always verify dynamically
- Create workflows without researching nodes first
- Skip validation before pushing
- Hardcode credentials in workflows
- Include `id` field for new workflows (let the push script handle it)
- Use sub-nodes as standalone nodes
- Invent parameter options — verify they exist in the node version

### Do
- **Run `validate_workflow` (MCP, strict profile) before EVERY push** — non-negotiable
- **Use `n8n_autofix_workflow` when validation fails** — auto-corrects typeVersions, expressions, connections
- **Use `get_node` to read full parameter schema** for every node you configure
- **Run `n8n_validate_workflow` (by ID) after push** to confirm instance-side validity
- Always start with MCP research (`search_nodes` + `get_node`)
- Pull existing workflows and grep for node types to see real configs
- Validate locally (`npm run validate`) AND via MCP before pushing
- Use environment variables for credentials
- Follow naming conventions
- Pull existing workflows to learn patterns
- Test workflows after pushing
- Use backward-connection loops for retries instead of duplicated chains

---

## Resources

- [n8n Documentation](https://docs.n8n.io)
- [n8n-mcp GitHub](https://github.com/czlonkowski/n8n-mcp) — MCP server for node docs
- [n8n-skills GitHub](https://github.com/czlonkowski/n8n-skills) — Original skills for n8n
- [SETUP.md](SETUP.md) — First-time setup guide (agent-readable)

---

## Process Summary

**Your workflow: Research > Design > Create > Local Validate > MCP Validate (strict) > Autofix > Push > Verify > Commit**

**Your #1 Rule:** Never push a workflow without passing `validate_workflow` (MCP, strict profile). This single step prevents most failures.

**When in doubt:** Pull a working workflow that uses the same node and copy its exact structure. Don't guess.
