# Glasp Highlights Auto Export — n8n Workflow

Automatically fetch your new [Glasp](https://glasp.co) highlights and export them to any destination — Notion, Slack, Google Sheets, Obsidian, or any API.

## How It Works

```
Schedule Trigger → Fetch & Filter Highlights → Has Highlights? → Format for Export → [Your Destination]
```

| Node | What it does |
|------|-------------|
| **Schedule Trigger** | Runs every 6 hours (configurable) |
| **Fetch & Filter Highlights** | Calls Glasp API with pagination, returns only new (unexported) documents |
| **Has Highlights?** | Filters out documents with no highlights |
| **Format for Export** | Outputs clean structure with plain text + Markdown formats |
| **[Your Destination]** | Connect any n8n node — Notion, Slack, Sheets, etc. |

## Setup

### Step 1: Get your Glasp Access Token

Go to **[glasp.co/settings/access_token](https://glasp.co/settings/access_token)** and copy your token.

### Step 2: Set your token

Open the **"Fetch & Filter Highlights"** Code node and replace `YOUR_GLASP_ACCESS_TOKEN` with your actual token:

```javascript
// Option A: Direct token (n8n Cloud users)
const token = 'paste-your-token-here';
```

**For self-hosted n8n users:** You can use an environment variable instead:

```javascript
// Option B: Environment variable (self-hosted)
const token = $env.GLASP_ACCESS_TOKEN;
```

> ⚠️ **Security:** If you fork this repo, never commit your actual token. The workflow JSON uses a placeholder `YOUR_GLASP_ACCESS_TOKEN` by default.

### Step 3: Import the workflow

| Method | How |
|--------|-----|
| **From file** | Download `glasp-highlights-export-workflow.json` → n8n → **⋯** → **Import from File** |
| **From URL** | n8n → **⋯** → **Import from URL** → paste the raw GitHub URL |

### Step 4: Connect your destination

Add a node after **"Format for Export"** to send highlights wherever you want. See examples below.

## Output Fields

Each item output by "Format for Export":

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

Add a **Notion** node → **Create Database Item**:
- Title → `{{ $json.title }}`
- URL → `{{ $json.url }}`
- Content → `{{ $json.highlightsMarkdown }}`
- Tags → `{{ $json.tags }}`

### Slack

Add a **Slack** node → **Send Message**:
```
📚 *{{ $json.title }}*
{{ $json.url }}
{{ $json.highlightCount }} highlights

{{ $json.highlightsText }}
```

### Google Sheets

Add a **Google Sheets** node → **Append Row**:
Map columns: `title`, `url`, `domain`, `highlightCount`, `highlightsText`, `createdAt`

### Webhook / Custom API

Add an **HTTP Request** node → POST `{{ $json }}` as JSON body.

### Obsidian (via Local REST API plugin)

Add an **HTTP Request** node → PUT to:
`http://localhost:27124/vault/Glasp/{{ $json.title }}.md`
Body: `{{ $json.highlightsMarkdown }}`

## Configuration

| Setting | Default | How to change |
|---------|---------|---------------|
| Schedule | Every 6 hours | Click Schedule Trigger node |
| First run lookback | 24 hours | Edit `24 * 60 * 60 * 1000` in Code node |
| Tracking cleanup | 30 days | Edit `30 * 24 * 60 * 60 * 1000` in Code node |
| Buffer time | 5 minutes | Edit `5 * 60 * 1000` in Code node |

## Glasp API

- **Endpoint:** `GET https://api.glasp.co/v1/highlights/export`
- **Auth:** Bearer token
- **Pagination:** cursor-based (`nextPageCursor` / `pageCursor`)
- **Rate limit:** 50 requests/minute
- **Docs:** [glasp.co/docs/api](https://glasp.co/docs/api)

## License

MIT
