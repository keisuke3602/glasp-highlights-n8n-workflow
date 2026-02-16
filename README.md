# Glasp Highlights Auto Export — n8n Workflow

Automatically fetch your new [Glasp](https://glasp.co) highlights and export them to any destination — Notion, Slack, Google Sheets, Obsidian, or any API.

## How It Works

```
Schedule Trigger → Fetch & Filter Highlights → Has Highlights? → Format for Export → [Your Destination]
```

1. **Schedule Trigger** — Runs every 6 hours (configurable)
2. **Fetch & Filter Highlights** — Calls the Glasp API with pagination, tracks already-exported documents, and returns only new ones
3. **Has Highlights?** — Filters out documents with no highlights
4. **Format for Export** — Transforms each document into a clean structure with both plain text and Markdown formats
5. **[Your Destination]** — Connect any n8n node to send highlights where you want

## Setup

### 1. Get your Glasp Access Token

Go to [glasp.co/settings/access_token](https://glasp.co/settings/access_token) and copy your token.

### 2. Set the environment variable in n8n

| Method | Instructions |
|--------|-------------|
| **n8n Cloud** | Settings → Environment Variables → Add `GLASP_ACCESS_TOKEN` |
| **Self-hosted** | Add `GLASP_ACCESS_TOKEN=your_token_here` to your `.env` file or Docker environment |

> **Security:** The token is never stored in the workflow JSON. It is read from the environment variable at runtime.

### 3. Import the workflow

- Download `glasp-highlights-export-workflow.json`
- In n8n, click **⋯** (top-right) → **Import from File** → select the JSON file
- Or **Import from URL** using the raw GitHub link

### 4. Connect your destination

Add a node after **"Format for Export"** to send highlights wherever you want.

## Output Format

Each item output by **"Format for Export"** has the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `documentId` | string | Glasp document ID |
| `title` | string | Article/page title |
| `url` | string | Original URL |
| `glasp_url` | string | Glasp page URL |
| `domain` | string | Source domain |
| `category` | string | `article`, `video`, `tweet`, `pdf`, `book` |
| `tags` | string[] | User tags |
| `author` | string | Author (if available) |
| `highlightCount` | number | Number of highlights |
| `highlightsText` | string | All highlights joined as plain text |
| `highlightsMarkdown` | string | Highlights formatted as Markdown |
| `highlights` | array | Individual highlights with `id`, `text`, `note`, `color`, `highlighted_at` |
| `createdAt` | string | ISO date — when the document was first highlighted |
| `updatedAt` | string | ISO date — last update |

## Destination Examples

### Notion

1. Add a **Notion** node → **Create Database Item**
2. Connect to your Notion database
3. Map fields:
   - Title → `{{ $json.title }}`
   - URL → `{{ $json.url }}`
   - Content → `{{ $json.highlightsMarkdown }}`
   - Tags → `{{ $json.tags }}`

### Slack

1. Add a **Slack** node → **Send Message**
2. Set channel and message:
   ```
   📚 *{{ $json.title }}*
   {{ $json.url }}
   {{ $json.highlightCount }} highlights

   {{ $json.highlightsText }}
   ```

### Google Sheets

1. Add a **Google Sheets** node → **Append Row**
2. Map columns: `title`, `url`, `domain`, `highlightCount`, `highlightsText`, `createdAt`

### Webhook / Custom API

1. Add an **HTTP Request** node → **POST**
2. Send `{{ $json }}` as JSON body to your endpoint

### Obsidian (via Local REST API plugin)

1. Add an **HTTP Request** node → **PUT**
2. Send `{{ $json.highlightsMarkdown }}` to `http://localhost:27124/vault/Glasp/{{ $json.title }}.md`

## Configuration

### Schedule Frequency

Click the **Schedule Trigger** node to change the interval. Default is every 6 hours.

### First Run Behavior

On the first execution, the workflow looks back **24 hours** to fetch recent highlights. Subsequent runs only fetch highlights updated since the last run.

### Deduplication

The workflow tracks exported document IDs in n8n's static data. Documents are only exported once. Tracking entries older than 30 days are automatically cleaned up.

## Glasp API Reference

- **Endpoint:** `GET https://api.glasp.co/v1/highlights/export`
- **Auth:** Bearer token
- **Params:** `updatedAfter` (ISO date), `pageCursor` (for pagination)
- **Rate limit:** 50 requests/minute
- **Docs:** [glasp.co/docs/api](https://glasp.co/docs/api)

## Requirements

- n8n (Cloud or self-hosted)
- Glasp account with access token
- n8n environment variable support

## License

MIT

## Links

- [Glasp](https://glasp.co) — Social Web Highlighter
- [Glasp API Docs](https://glasp.co/docs/api)
- [n8n](https://n8n.io) — Workflow Automation
- [n8n Environment Variables](https://docs.n8n.io/hosting/configuration/environment-variables/)
