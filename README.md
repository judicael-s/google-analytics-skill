# Google Analytics 4 — MCP Server & AI Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js 20+](https://img.shields.io/badge/Node.js-20%2B-green.svg)](https://nodejs.org/)
[![MCP](https://img.shields.io/badge/MCP-compatible-purple.svg)](https://modelcontextprotocol.io/)

Query your GA4 property from any MCP-compatible client — traffic reports, top pages, active users, events, and user behavior. Works with Claude Desktop, Claude Code, Cursor, Windsurf, Cline, and any tool supporting the [Model Context Protocol](https://modelcontextprotocol.io/).

## How It Works

This project packages the [`mcp-server-google-analytics`](https://github.com/ruchernchong/mcp-server-google-analytics) MCP server with ready-to-use configuration and an optional Claude Code skill file for guided workflows.

The MCP server connects to the GA4 Data API using a Google service account and exposes 5 tools that any MCP client can call.

## Prerequisites

- **Node.js 20+**
- A Google Cloud project with the **Analytics Data API** enabled
- A service account with **Viewer** access to your GA4 property

## Setup

### Step 1: Create a Google Cloud Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a project (or use an existing one)
2. Enable the **Google Analytics Data API** (APIs & Services > Library > search "Analytics Data API")
3. Create a service account (IAM & Admin > Service Accounts > Create)
4. Create and download a JSON key for the service account
5. In **Google Analytics** > Admin > Property Access Management, add the service account email as a **Viewer**

### Step 2: Add MCP Server to Your Client

Create or edit your MCP configuration file:

<details open>
<summary><strong>Claude Desktop</strong> — <code>claude_desktop_config.json</code></summary>

```json
{
  "mcpServers": {
    "google-analytics": {
      "command": "npx",
      "args": ["-y", "mcp-server-google-analytics"],
      "env": {
        "GOOGLE_CLIENT_EMAIL": "your-service-account@project.iam.gserviceaccount.com",
        "GOOGLE_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----",
        "GA_PROPERTY_ID": "123456789"
      }
    }
  }
}
```

Config location: `%APPDATA%\Claude\claude_desktop_config.json` (Windows) or `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)

</details>

<details>
<summary><strong>Claude Code</strong> — <code>.mcp.json</code> (project root)</summary>

```json
{
  "mcpServers": {
    "google-analytics": {
      "command": "npx",
      "args": ["-y", "mcp-server-google-analytics"],
      "env": {
        "GOOGLE_CLIENT_EMAIL": "your-service-account@project.iam.gserviceaccount.com",
        "GOOGLE_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----",
        "GA_PROPERTY_ID": "123456789"
      }
    }
  }
}
```

</details>

<details>
<summary><strong>Cursor / Windsurf / Cline</strong></summary>

Use the same JSON structure above in your editor's MCP configuration. Check your editor's docs for the config file location:
- **Cursor:** `.cursor/mcp.json`
- **Windsurf:** `~/.codeium/windsurf/mcp_config.json`
- **Cline:** VS Code settings > Cline MCP Servers

</details>

> **Tip:** Your `GA_PROPERTY_ID` is the numeric ID from GA4 Admin > Property Settings (e.g. `123456789`). Do not include the `properties/` prefix.

### Step 3: Verify

Restart your MCP client and ask: *"Show me my active users for the last 7 days"*

## Available Tools

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `runReport` | Flexible custom reports with any dimensions/metrics | `startDate`, `endDate`, `dimensions`, `metrics`, `dimensionFilter` |
| `getPageViews` | Page view metrics by dimension | `startDate`, `endDate`, `dimensions` |
| `getActiveUsers` | Active users over time | `startDate`, `endDate` |
| `getEvents` | Event analysis | `startDate`, `endDate`, `eventName` |
| `getUserBehavior` | Session duration, bounce rate, engagement | `startDate`, `endDate` |

## Usage Examples

Ask your AI assistant in natural language:

- "Show me my top 10 pages by views this month"
- "How many active users did I have last week vs the week before?"
- "What events are firing most on my site?"
- "Break down my traffic by country for the last 30 days"
- "What's my bounce rate trend over the past 3 months?"
- "Compare desktop vs mobile sessions this quarter"
- "Which traffic sources are driving the most conversions?"

## Common Dimensions & Metrics Reference

### Dimensions

| Dimension | Description |
|-----------|-------------|
| `date` | Date in YYYYMMDD format |
| `city` | User's city |
| `country` | User's country |
| `deviceCategory` | desktop, mobile, or tablet |
| `sessionSource` | Traffic source (google, direct, etc.) |
| `sessionMedium` | Traffic medium (organic, cpc, referral, etc.) |
| `pagePath` | URL path of the page |
| `pageTitle` | Title of the page |
| `eventName` | Name of the event |
| `browser` | User's browser |
| `operatingSystem` | User's OS |

### Metrics

| Metric | Description |
|--------|-------------|
| `activeUsers` | Users who had an engaged session |
| `newUsers` | First-time users |
| `sessions` | Total sessions |
| `screenPageViews` | Total page views |
| `eventCount` | Total events fired |
| `conversions` | Conversion events |
| `totalRevenue` | Total revenue (if ecommerce is set up) |
| `bounceRate` | Percentage of non-engaged sessions |
| `averageSessionDuration` | Average session length in seconds |
| `engagedSessions` | Sessions with engagement |

## Use as a Claude Code Skill

This repo includes a `SKILL.md` file that turns it into a [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) skill with 5 guided workflows and an interactive setup assistant.

```bash
mkdir -p ~/.claude/skills/google-analytics
cp SKILL.md ~/.claude/skills/google-analytics/SKILL.md
```

The skill will walk users through MCP server setup if it's not already configured.

## Alternative MCP — Composio / Rube (Freemium)

If you prefer a managed solution that handles OAuth without a service account:

- [Composio Rube MCP](https://rube.app/mcp) — Free tier: 1,000 requests/day. Pro: $25/month.
- See [Composio pricing](https://composio.dev/pricing) for details.

## Credits

- **MCP Server:** [mcp-server-google-analytics](https://github.com/ruchernchong/mcp-server-google-analytics) by [ruchernchong](https://github.com/ruchernchong)
- **Alternative MCP:** [Composio/Rube](https://composio.dev)

## Part of the Marketing Suite

| Tool | Repository |
|------|------------|
| Google Trends | [judicael-s/google-trends-skill](https://github.com/judicael-s/google-trends-skill) |
| **Google Analytics** | **This repo** |
| Google Search Console | [judicael-s/google-search-console-skill](https://github.com/judicael-s/google-search-console-skill) |

## License

[MIT](LICENSE)
