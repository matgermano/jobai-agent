# MCP — Model Context Protocol

## What MCP is

MCP (Model Context Protocol) is the protocol that lets Claude operate external systems as if they were native tools. With MCP, Claude doesn't just describe how to update Notion — it calls `notion.update_page()` and the page updates. It doesn't just describe how to send an email — it calls `gmail.send_email()` and the email is sent.

**Without MCP:**
```
User: "Update my Kanban card to Done"
Claude: "Here's how you would update it in Notion: go to the page, click..."
```

**With MCP:**
```
User: "Update my Kanban card to Done"
Claude: calls notion-update-page() → card status = Done ✓
```

MCP turns Claude from a text generator into an agent that acts in the world.

---

## How it works technically

MCP defines a communication protocol between:
- **Host** (Claude Code) — the agent that wants to use tools
- **Server** (Notion, Gmail, GitHub, etc.) — the system being controlled

The MCP server exposes a set of **tools** with JSON schemas. Claude calls these tools exactly like it calls native tools (Read, Write, Bash). The tools internally make API calls to the external service and return structured results.

```
Claude → mcp__claude_ai_Notion__notion-update-page({page_id, data})
       → Notion API
       → Page updated, confirmation returned to Claude
```

The tool names follow the pattern: `mcp__[connector_name]__[tool_name]`

---

## How to connect a service

### Via claude.ai connectors (recommended — OAuth, no API key required)

1. Go to `claude.ai/customize/connectors`
2. Find the connector (Notion, Gmail, GitHub, Slack, Linear, etc.)
3. Authorize via OAuth — Claude receives a temporary access token, never your password
4. The connector becomes automatically available in all Claude Code local sessions and in Routines

### Via manual MCP server (for custom or unsupported services)

For services without an official connector, you can build a custom MCP server and register it in `.claude/settings.json`. This is an advanced path — for most common services, the claude.ai connectors already exist.

---

## Active integrations in jobAI

### Notion
**Connector UUID:** `cf2d86aa-dc6e-46ab-95cf-1de1ab3020c9`
**Connected via:** claude.ai OAuth
**Used for:** Kanban board management via `/updatekanban` command

**Available tools:**
- `notion-search` — finds pages and databases by text query
- `notion-fetch` — reads the full content of a page
- `notion-update-page` — updates page properties (e.g., Status field)
- `notion-create-pages` — creates new pages
- `notion-create-database` — creates new databases
- `notion-get-comments` / `notion-create-comment` — reads and adds comments

**How the `/updatekanban` command uses it:**
```
/updatekanban "Card 1.4" Done

→ Claude calls notion-search("Card 1.4") in the "jobAI Roadmap" database
→ Gets the page_id from the result
→ Calls notion-update-page(page_id, {Status: "Done"})
→ Confirms the update
```

**How Routines access Notion:** The `connector_uuid` is listed in the `mcp_connections` field of each Routine config. All three jobAI routines have Notion connected, even if they don't always use it — it's available if needed.

---

### Gmail
**Connected via:** claude.ai OAuth
**Used for:** Immediate email alerts when a job scores ≥ 9/10

**How it's triggered in hunt-jobs.md:**
```
After scoring all jobs:
If any job has fit_score >= 9:
  → Send email to the address in config.json
  → Subject: "jobAI Alert: [Job Title] at [Company] — Score [X]/10"
  → Body: title, company, URL, fit_score, key requirements, found_at
```

**Why email instead of waiting for the Monday report:**
A 9/10 job may close within 24-48 hours. The weekly report arrives Monday morning — that's too late if the job was posted Thursday. The Gmail alert ensures immediate notification for top-scoring matches.

**Note on Gmail in Routines:** Gmail MCP is available in the Daily Job Hunter routine because it's connected at the claude.ai level. The routine's prompt instructs the agent to use it conditionally — only when a high-fit job is found.

---

## Permission model and security

**OAuth tokens:** When you connect a service via claude.ai, it uses OAuth. Claude receives a scoped access token — it never sees your password. Revoking the connector in claude.ai immediately invalidates the token.

**`permitted_tools` per routine:** Each `mcp_connection` in a Routine can specify `permitted_tools` to limit which MCP tools the agent can call. Leaving it empty (`[]`) means all tools for that connector are available. For production hardening, you can restrict — e.g., the Weekly Report can read Notion but doesn't need to create pages.

**Principle of least privilege:** Connect only what the agent actually needs. An agent doing CV analysis doesn't need Gmail access. An agent sending notifications doesn't need Notion access.

---

## MCP integration roadmap for jobAI

### Phase 2 — LinkedIn analysis
Compare your LinkedIn headline, about section, and experience bullets against the top keywords from `gap_report.md`. Suggest specific copy improvements.
- **Requires:** LinkedIn MCP connector (check `claude.ai/customize/connectors` for availability)

### Phase 3 — Chrome MCP (browser automation)
The agent opens a browser, navigates to an ATS job application (Greenhouse, Lever, Ashby), reads `application_profile.json`, and fills in all form fields. Pauses before submitting for your review.
- **Requires:** "Claude in Chrome" browser extension (currently in beta)
- **Risk level:** High — browser automation can trigger bot detection. Always review before submitting.

### How to add a new connector to an existing Routine

1. Connect the service at `claude.ai/customize/connectors`
2. Note the `connector_uuid` from the connectors page
3. In Claude Code, run `/schedule` → update the target routine → add the connector
4. The update takes effect on the next scheduled run

---

## Why MCP matters beyond this project

Before MCP, integrating LLMs with external systems required building custom wrappers for each service — managing auth flows, serialization, error handling, rate limits. Each integration was one-off engineering work.

MCP standardizes this. A well-built MCP server can be used by any compatible host. It's the same conceptual shift that HTTP made for the web — a standard protocol that decoupled clients from servers, enabling a market of interoperable tools.

For a Product Manager or AI Builder, understanding MCP is understanding how agents cross the boundary from "generates text about the world" to "acts in the world." The difference between a chatbot and an autonomous agent is largely in what MCP connections it has available.
