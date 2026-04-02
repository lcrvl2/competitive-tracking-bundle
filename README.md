# Competitive Tracking Bundle

Two n8n workflows + one sub-workflow to monitor your competitors automatically and get notified in Slack.

## What's inside

| Workflow | File | What it does |
|----------|------|-------------|
| Competitor Sitemap Monitoring | `wf1-competitor-sitemap-monitoring.json` | Scans competitor sitemaps daily. Detects new pages published. Sends a Slack digest. |
| Page Change Tracker | `wf2-page-change-tracker.json` | Monitors specific pages (pricing, features...) weekly. Detects text changes. Generates a visual diff report (PDF). |
| Diff Report Generator | `wf3-diff-report-generator.json` | Sub-workflow called by WF2. Converts JSON diffs to styled HTML then PDF. |

## How it works

### WF1 -- Competitor Sitemap Monitoring

Runs daily. Reads your competitor domains from a Google Sheet. For each domain, fetches the sitemap XML, handles nested sitemap indexes, extracts all URLs. Compares against the previous snapshot stored in an n8n Data Table. New URLs trigger a Slack notification with the list.

**Flow:**
```
Schedule (daily) -> Read domains (Google Sheets) -> Fetch sitemaps -> Parse XML -> Detect new URLs -> Update snapshot (Data Table) -> Slack alert
```

### WF2 -- Page Change Tracker

Runs weekly. Loops through a list of specific competitor URLs (pricing pages, feature pages, etc.). Uses [Firecrawl](https://firecrawl.dev)'s change tracking API to detect content changes. When a change is found, triggers the Diff Report Generator sub-workflow, which creates a styled PDF report and posts it to Slack.

**Flow:**
```
Schedule (weekly) -> Loop URLs -> Firecrawl change tracking -> If changed -> Generate diff report (PDF) -> Slack alert
```

### WF3 -- Diff Report Generator

Sub-workflow called by WF2. Takes JSON diff data, uses GPT-4.1-mini to convert it into semantic HTML with proper diff styling (additions in green, deletions in red), then converts to PDF. The HTML template includes responsive CSS for git-diff blocks, title change tables, and clean typography.

## Prerequisites

| Service | Used by | Purpose |
|---------|---------|---------|
| [n8n](https://n8n.io) | All | Workflow automation platform (self-hosted or cloud) |
| [Google Sheets](https://sheets.google.com) | WF1 | Stores the list of competitor domains + sitemap URLs |
| n8n Data Tables | WF1 | Stores URL snapshots for diff comparison |
| [Firecrawl](https://firecrawl.dev) | WF2 | Change tracking API for web pages |
| [OpenAI](https://openai.com) | WF3 | GPT-4.1-mini for JSON to HTML conversion |
| [htmlcsstopdf](https://htmlcsstopdf.com) | WF3 | HTML to PDF conversion (n8n community node) |
| [Slack](https://slack.com) | WF1, WF2 | Notification channel |

## Setup

### 1. Import workflows

In n8n, go to **Workflows > Import from file** and import all 3 JSON files.

### 2. Create a Google Sheet (for WF1)

Create a Google Sheet with a tab named "Configuration" containing columns:
- `Sitemap` -- The full sitemap URL for each competitor (e.g., `https://competitor.com/sitemap.xml`)

### 3. Create an n8n Data Table (for WF1)

Create a Data Table named `sitemap_snapshots` with columns:
- `site` (string) -- The domain
- `urls_json` (string) -- JSON array of all URLs (managed automatically)

### 4. Configure credentials

Set up these credentials in n8n:
- **Google Sheets OAuth2** -- For reading the competitor list
- **Slack API** -- For sending notifications
- **Firecrawl API** (HTTP Header Auth) -- For change tracking
- **OpenAI API** -- For the diff report generator
- **htmlcsstopdf API** -- For PDF generation

### 5. Customize

- **WF1:** Update the Google Sheet with your competitors' sitemap URLs
- **WF2:** Edit the "Build URL list" node with the specific pages you want to track
- **WF1:** Update the `EXCLUDE_FROM_SLACK` list in "Slack message builder" to exclude your own domain
- **WF2:** Link the "Generate Report" node to your imported WF3 (Diff Report Generator)
- Update Slack channel names in both workflows

### 6. Activate

Turn on WF1 and WF2. WF3 stays inactive (it's called as a sub-workflow).

## Community node required

WF3 uses the `n8n-nodes-htmlcsstopdf` community node. Install it in n8n:
**Settings > Community nodes > Install > `n8n-nodes-htmlcsstopdf`**

## License

MIT

---

Built by [Lucas Carval](https://www.linkedin.com/in/lucascarval/) -- Content Engineer for B2B
