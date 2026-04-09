# Google Analytics Skill for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Node.js 20+](https://img.shields.io/badge/Node.js-20%2B-green.svg)](https://nodejs.org/)
[![MCP](https://img.shields.io/badge/MCP-compatible-purple.svg)](https://modelcontextprotocol.io/)

A Claude Code skill for GA4 reporting. Ask Claude about your website traffic, audience, top pages, events, and more — powered by the Google Analytics Data API via MCP.

---

## How It Works

This skill connects Claude Code to your GA4 property through the `mcp-server-google-analytics` MCP server. Claude can then run reports, check page views, analyze user behavior, and track events using natural language.

The skill file (`SKILL.md`) teaches Claude *how* to use the GA4 tools effectively — which tool to pick, what parameters to pass, and how to present the results.

---

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Node.js 20+
- A Google Cloud project with the **Analytics Data API** enabled
- A service account with **Viewer** access to your GA4 property

---

## Setup — MCP Server (Free & Open Source)

The primary method uses [`mcp-server-google-analytics`](https://github.com/ruchernchong/mcp-server-google-analytics) by [ruchernchong](https://github.com/ruchernchong).

### Step 1: Create a Google Cloud Service Account

1. Go to [Google Cloud Console](https://console.cloud.google.com/) and create a project (or use an existing one)
2. Enable the **Google Analytics Data API** (APIs & Services > Library > search "Analytics Data API")
3. Create a service account (IAM & Admin > Service Accounts > Create)
4. Create and download a JSON key for the service account
5. In **Google Analytics** > Admin > Property Access Management, add the service account email as a **Viewer**

### Step 2: Add MCP Server to Your Claude Code Project

Create or edit `.mcp.json` in your project root:

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

> **Tip:** Your `GA_PROPERTY_ID` is the numeric ID from GA4 Admin > Property Settings (e.g. `123456789`). Do not include the `properties/` prefix.

### Step 3: Install the Skill

Copy `SKILL.md` to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/google-analytics
cp SKILL.md ~/.claude/skills/google-analytics/SKILL.md
```

Restart Claude Code. You can now ask questions like "Show me my top pages this month" and Claude will query GA4 automatically.

---

## Alternative Setup — Composio/Rube MCP (Freemium)

If you prefer a managed solution that handles OAuth and authentication for you, [Composio Rube MCP](https://rube.app/mcp) offers a Google Analytics integration with no service account setup required.

- **Free tier:** 1,000 requests/day
- **Pro:** $25/month (higher limits, priority support)

See [Composio pricing](https://composio.dev/pricing) and [Rube MCP docs](https://rube.app/mcp) for details.

---

## Available Tools

| Tool | Description | Key Parameters |
|------|-------------|----------------|
| `runReport` | Flexible custom reports with any dimensions/metrics | `startDate`, `endDate`, `dimensions`, `metrics`, `dimensionFilter` |
| `getPageViews` | Page view metrics by dimension | `startDate`, `endDate`, `dimensions` |
| `getActiveUsers` | Active users over time | `startDate`, `endDate` |
| `getEvents` | Event analysis | `startDate`, `endDate`, `eventName` |
| `getUserBehavior` | Session duration, bounce rate, engagement | `startDate`, `endDate` |

---

## Usage Examples

Just ask Claude in natural language:

- "Show me my top 10 pages by views this month"
- "How many active users did I have last week vs the week before?"
- "What events are firing most on my site?"
- "Break down my traffic by country for the last 30 days"
- "What's my bounce rate trend over the past 3 months?"
- "Compare desktop vs mobile sessions this quarter"
- "Which traffic sources are driving the most conversions?"

---

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

---

## Credits

- **MCP Server:** [mcp-server-google-analytics](https://github.com/ruchernchong/mcp-server-google-analytics) by [ruchernchong](https://github.com/ruchernchong)
- **Alternative MCP:** [Composio/Rube](https://composio.dev)

---

## Part of the Marketing Suite

| Skill | Repository |
|-------|------------|
| Google Trends | [judicael-s/google-trends-skill](https://github.com/judicael-s/google-trends-skill) |
| **Google Analytics** | **This repo** |
| Google Search Console | [judicael-s/google-search-console-skill](https://github.com/judicael-s/google-search-console-skill) |

---

## License

[MIT](LICENSE)
