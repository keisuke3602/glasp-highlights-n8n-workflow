# Glasp Highlights Auto Export -- n8n Workflow

Automatically fetch your new [Glasp](https://glasp.co) highlights and export them to any destination -- Notion, Slack, Google Sheets, or any API.

## How It Works

```
Schedule Trigger --> Prepare Parameters --> Glasp API --> Filter & Format --> [Your Destination]
```

| Node | What it does |
|------|-------------|
| **Schedule Trigger** | Runs every 6 hours (configurable) |
| **Prepare Parameters** | Calculates time range and tracks exported docs |
| **Glasp API** | HTTP Request node with Bearer Auth + pagination |
| **Filter & Format** | Returns only new docs, outputs plain text + Markdown |
| **[Your Destination]** | Connect any n8n node -- Notion, Slack, Sheets, etc. |

## Setup

### Step 1: Import the workflow

| Method | How |
|--------|-----|
| **From file** | Download `glasp-highlights-export-workflow.json` then in n8n: **...** menu --> **Import from File** |
| **From URL** | In n8n: **...** menu --> **Import from URL** --> paste the raw GitHub URL |

### Step 2: Get your Glasp Access Token

Go to **[glasp.co/settings/access_token](https://glasp.co/settings/access_token)** and copy your token.

### Step 3: Create a credential in n8n

1. Open the **Glasp API** node
2. In **Authentication**, select **Predefined Credential Type**
3. Credential Type: **Bearer Auth**
4. Click **Create New Credential**
5. Paste your Glasp token as the **Token** value
6. Save

> **Security:** The token is stored in n8n's encrypted credential store, never in the workflow JSON. Safe to share and commit to Git.

### Step 4: Connect your destination

Add a node after **"Filter & Format"** to send highlights wherever you want. See examples below.

## Output Fields

Each item output by "Filter & Format":

| Field | Type | Description |
|-------|------|-------------|
| `documentId` | string | Glasp document ID |
| `title` | string | Article/page title |
| `url` | string | Original URL |
| `glasp_url` | string | Glasp page URL |
| `domain` | string | Source domain |
| `category` | string | `article` / `video` / `tweet` / `pdf` / `book` |
| `tags` | string[] | User tags |
| `author` | string | Author if available |
| `highlightCount` | number | Number of highlights |
| `highlightsText` | string | All highlights as plain text |
| `highlightsMarkdown` | string | Formatted as Markdown |
| `highlights` | array | Objects with `id`, `text`, `note`, `color`, `highlighted_at` |
| `createdAt` | string | ISO date |
| `updatedAt` | string | ISO date |

## Destination Examples

### Notion

Add a **Notion** node --> **Create Database Item**:
- Title: `{{ $json.title }}`
- URL: `{{ $json.url }}`
- Content: `{{ $json.highlightsMarkdown }}`
- Tags: `{{ $json.tags }}`

### Slack

Add a **Slack** node --> **Send Message**:
```
*{{ $json.title }}*
{{ $json.url }}
{{ $json.highlightCount }} highlights

{{ $json.highlightsText }}
```

### Google Sheets

Add a **Google Sheets** node --> **Append Row**:
Map columns: `title`, `url`, `domain`, `highlightCount`, `highlightsText`, `createdAt`

### Webhook / Custom API

Add an **HTTP Request** node --> POST `{{ $json }}` as JSON body.

## Configuration

| Setting | Default | How to change |
|---------|---------|---------------|
| Schedule | Every 6 hours | Click Schedule Trigger node |
| First run lookback | 24 hours | Edit `24 * 60 * 60 * 1000` in Prepare Parameters |
| Tracking cleanup | 30 days | Edit `30 * 24 * 60 * 60 * 1000` in Filter & Format |
| Buffer time | 5 minutes | Edit `5 * 60 * 1000` in Prepare Parameters |

## How Deduplication Works

The workflow tracks exported document IDs in n8n's static data storage. Each document is only exported once. Tracking entries older than 30 days are automatically cleaned up to prevent unbounded growth.

On the first run, the workflow looks back 24 hours. Subsequent runs only fetch highlights updated since the last run, with a 5-minute overlap buffer to avoid missing any.

## Glasp API

- **Endpoint:** `GET https://api.glasp.co/v1/highlights/export`
- **Auth:** Bearer token
- **Pagination:** cursor-based (`nextPageCursor` / `pageCursor`)
- **Rate limit:** 50 requests/minute
- **Docs:** [glasp.co/docs/api](https://glasp.co/docs/api)

## Tutorial

For a step-by-step guide with screenshots, see the full tutorial:
[How to Auto-Export Glasp Highlights with n8n](https://glasp.co/posts/how-to-autoexport-glasp-highlights-with-n8n)

## Requirements

- n8n (Cloud or self-hosted)
- Glasp account with access token

## License

MIT
