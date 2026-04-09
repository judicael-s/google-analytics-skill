---
name: google-analytics
description: "Query Google Analytics 4 data via MCP. Use when the user asks about website traffic, page views, active users, events, user behavior, bounce rate, session duration, audience demographics, traffic sources, or any GA4 reporting. Trigger on: traffic analysis, analytics report, page performance, user engagement, conversion tracking, audience insights."
requires:
  mcp: [google-analytics]
license: MIT
---

# Google Analytics 4 Reporting

Query your GA4 property using natural language — traffic, pages, users, events, and behavior metrics.

## Prerequisites & Setup

**Before any query, verify the MCP server is available.** Check if these tools exist: `runReport`, `getPageViews`, `getActiveUsers`, `getEvents`, `getUserBehavior`.

**If the tools are NOT available, walk the user through setup:**

### Step 1: Install the MCP Server

Ask the user to create or edit `.mcp.json` in their project root:

```json
{
  "mcpServers": {
    "google-analytics": {
      "command": "npx",
      "args": ["-y", "mcp-server-google-analytics"],
      "env": {
        "GOOGLE_CLIENT_EMAIL": "<service-account-email>",
        "GOOGLE_PRIVATE_KEY": "<private-key-from-json>",
        "GA_PROPERTY_ID": "<numeric-property-id>"
      }
    }
  }
}
```

### Step 2: Get Credentials

Guide the user through these steps:

1. **Google Cloud Console** — Go to https://console.cloud.google.com/
2. **Create or select a project**
3. **Enable the Analytics Data API** — APIs & Services > Library > search "Analytics Data API" > Enable
4. **Create a service account** — IAM & Admin > Service Accounts > Create Service Account
5. **Generate a JSON key** — click the service account > Keys > Add Key > JSON > Download
6. **Extract values from the JSON file:**
   - `client_email` → use as `GOOGLE_CLIENT_EMAIL`
   - `private_key` → use as `GOOGLE_PRIVATE_KEY`
7. **Grant access in GA4** — Google Analytics > Admin > Property Access Management > Add the service account email as **Viewer**
8. **Get property ID** — Google Analytics > Admin > Property Settings > copy the numeric Property ID (e.g. `518920216`)

### Step 3: Verify

Ask the user to restart Claude Code, then test with: "Show me my active users for the last 7 days"

If the MCP server fails to connect, check:
- Node.js 20+ is installed
- The private key includes the full `-----BEGIN PRIVATE KEY-----` header/footer
- The service account email has Viewer access in GA4
- The property ID is numeric only (no `properties/` prefix)

## Workflow

Follow these steps for every GA4 request.

### Step 1: Parse the Request

Identify what the user wants and map it to the right workflow:

| User Intent | Workflow | Primary Tool |
|-------------|----------|-------------|
| Traffic overview, trends over time, general stats | Traffic Overview | `runReport` |
| Top pages, page performance, content analysis | Top Pages | `getPageViews` or `runReport` |
| Audience breakdown by country, device, browser | Audience Analysis | `runReport` |
| Event tracking, what events fire, event counts | Event Tracking | `getEvents` |
| Bounce rate, session duration, engagement metrics | User Behavior | `getUserBehavior` |
| Custom or complex queries combining multiple dimensions | Custom Report | `runReport` |

Determine the date range from the user's request. Use these conventions:
- "this month" = startDate: first day of current month, endDate: today
- "last week" = startDate: 7 days ago, endDate: yesterday
- "last 30 days" = startDate: `30daysAgo`, endDate: `today`
- "last 3 months" = startDate: `90daysAgo`, endDate: `today`
- If no date range is specified, default to the last 28 days

### Step 2: Execute the Query

#### Workflow 1: Traffic Overview

**When to use:** User asks about overall traffic, sessions, users, or general performance trends.

**Tool:** `runReport`

**Parameters:**
- `startDate` / `endDate`: from the user's request
- `dimensions`: `["date"]`
- `metrics`: `["activeUsers", "sessions", "screenPageViews", "bounceRate"]`

**Presentation:** Show a summary of totals, then a trend table with key dates (weekly or daily depending on range length). Highlight any notable spikes or dips.

#### Workflow 2: Top Pages

**When to use:** User asks about best-performing pages, content analysis, or page-level metrics.

**Tool:** `getPageViews` with dimensions `["pagePath"]`, or `runReport` with dimensions `["pagePath", "pageTitle"]` and metrics `["screenPageViews", "activeUsers", "bounceRate"]`.

**Parameters:**
- `startDate` / `endDate`: from the user's request
- `dimensions`: `["pagePath"]` (add `pageTitle` if the user wants readable names)

**Presentation:** Ranked table of top pages. Default to top 10 unless the user specifies otherwise.

#### Workflow 3: Audience Analysis

**When to use:** User asks about who their visitors are — geography, devices, browsers, traffic sources.

**Tool:** `runReport`

**Parameters:**
- `startDate` / `endDate`: from the user's request
- `dimensions`: pick from `["country"]`, `["deviceCategory"]`, `["browser"]`, `["sessionSource", "sessionMedium"]` based on what the user asks
- `metrics`: `["activeUsers", "sessions", "screenPageViews"]`

**Presentation:** Ranked table by the primary dimension. For traffic sources, combine source/medium into a single column.

#### Workflow 4: Event Tracking

**When to use:** User asks about events, conversions, or specific user actions.

**Tool:** `getEvents`

**Parameters:**
- `startDate` / `endDate`: from the user's request
- `eventName`: if the user specifies a particular event, pass it here; otherwise omit to get all events

**Presentation:** Ranked list of events by count. Group standard GA4 events (page_view, session_start, first_visit, scroll, click) separately from custom events if both are present.

#### Workflow 5: User Behavior

**When to use:** User asks about engagement, bounce rate, session duration, or user quality metrics.

**Tool:** `getUserBehavior`

**Parameters:**
- `startDate` / `endDate`: from the user's request

**Presentation:** Key metrics summary with context (e.g., "Bounce rate of 45% is within the typical range for content sites"). Compare to previous period if the user asks for trend context.

### Step 3: Analyze & Present

Use this output template:

```
## GA4 Report: [Report Type] — [Date Range]

### Key Findings
- [2-3 bullet points with the most important takeaways]

### [Report-Specific Section]
[Data tables, ranked lists, or trend summaries as appropriate]

| [Dimension] | [Metric 1] | [Metric 2] | ... |
|-------------|------------|------------|-----|
| ...         | ...        | ...        | ... |

### Recommendations
- [2-4 actionable insights based on the data]

### Notes
- Data source: Google Analytics 4
- Property ID: [from the MCP connection]
- Date range: [startDate] to [endDate]
```

### Step 4: Handle Follow-ups

After presenting results, the user may ask:
- **Drill down:** "Break that down by device" — run a new query adding the dimension
- **Compare periods:** "Compare to last month" — run the same query for both periods and show a comparison
- **Filter:** "Only show organic traffic" — use `dimensionFilter` in `runReport`
- **Export:** "Give me this as a CSV" — format the data as CSV in a code block

## Common Pitfalls

- **Date format:** Use `YYYY-MM-DD` (e.g., `2025-01-15`) or relative strings like `7daysAgo`, `30daysAgo`, `today`, `yesterday`
- **Property ID:** Numeric only (e.g., `123456789`). Do not include the `properties/` prefix — this MCP server handles that internally
- **Dimension/metric compatibility:** Not all combinations work. If a query fails, simplify by removing dimensions or switching metrics. Common incompatible combo: `totalRevenue` with non-ecommerce properties
- **Rate limits:** The GA4 Data API has quotas. If you hit a rate limit, wait a moment and retry. Avoid running many back-to-back queries unnecessarily
- **Sampling:** Large date ranges or high-cardinality dimensions may return sampled data. Note this in your response if the API indicates sampling was applied
- **Timezone:** GA4 data uses the property's configured timezone, not the user's local timezone

## Limitations

- **Historical data:** Only data that exists in the GA4 property is available. If the property was recently created, historical data may be limited.
- **Real-time data:** These tools query processed data, not real-time. There is typically a 24-48 hour delay for the most recent data.
- **Custom dimensions/metrics:** Only standard GA4 dimensions and metrics are documented here. Custom ones work but you need to know the exact API name.
- **Data retention:** GA4 free properties retain detailed data for 2 or 14 months (configurable). Aggregated data is available longer.
- **No write access:** This skill is read-only. It cannot create goals, modify property settings, or configure tracking.
