# ikman-n8n-scraper

n8n workflow that scrapes [ikman.lk](https://ikman.lk) listings (e.g. cars, motorbikes), extracts structured data with Google Gemini, and appends new rows to a Google Sheet with duplicate detection.

## What it does

- **Trigger**: Runs on a schedule (default: hourly).
- **Scraping**: Builds paginated listing URLs (10 pages), fetches each page with a 4-second throttle to avoid hammering the site.
- **Extraction**: Strips HTML from the response and uses an **Information Extractor** (LangChain + Google Gemini) to parse ad fields (id, slug, title, price, location, category, etc.) from the text.
- **Deduplication**: Reads existing rows from a Google Sheet, merges with newly extracted rows, filters by ad `id`, and appends only rows that are not already in the sheet.
- **Output**: Appends new listings to the configured Google Sheet (auto-mapped columns).

## Prerequisites

- [n8n](https://n8n.io) (self-hosted or cloud).
- **Google Gemini (PaLM) API** key for the Information Extractor node.
- **Google Sheets OAuth2** credentials so n8n can read/append to your spreadsheet.
- A **Google Sheet** with a header row (or empty sheet; the workflow uses auto-map). The workflow uses column `id` for duplicate matching.

## Setup

### 1. Import the workflow

- In n8n: **Workflows** → **Import from File** → choose `Ikman cars.json` from this repo.

### 2. Configure credentials

- **Google Gemini Chat Model**  
  - Open the node “Google Gemini Chat Model” and add/sign in with a **Google PaLM / Gemini API** credential (your API key).
- **Google Sheets (Read + Append)**  
  - Open **Google Sheets Read** and **Google Sheets Append** and add/sign in with a **Google Sheets OAuth2** credential so n8n can access your spreadsheet.

### 3. Set your Google Sheet ID

- In both **Google Sheets Read** and **Google Sheets Append** nodes, replace `YOUR_GOOGLE_SHEET_ID` with your spreadsheet ID.  
  - Example: for `https://docs.google.com/spreadsheets/d/1i5Ox_Kj658IKX66R_dV7oP1r_y0ItcG_9-rfzkiRu98/edit`, the ID is `1i5Ox_Kj658IKX66R_dV7oP1r_y0ItcG_9-rfzkiRu98`.
- Pick the correct sheet (e.g. `Sheet1` / `gid=0`) in the “Sheet” dropdown if needed.

### 4. (Optional) Change category or base URL

- Default base URL is set in **Set base URL** via workflow variable `ikmanBaseUrl`, falling back to:  
  `https://ikman.lk/en/ads/sri-lanka/cars`
- To scrape another category (e.g. motorbikes), set **Workflow variables** (or edit the Set node) so `ikmanBaseUrl` is e.g.  
  `https://ikman.lk/en/ads/sri-lanka/motorbikes-scooters`

### 5. Activate

- Save the workflow and turn it **Active** so the schedule trigger runs.

## Workflow overview

```
Schedule Trigger
       │
       ├──► Set base URL ──► Generate page URLs ──► Init static ──► Split In Batches ──┐
       │                                                                               │
       │    (per batch)   Wait (4s) ──► HTTP Request ──► Code (strip HTML)             │
       │                                        │                                      │
       │                                        ▼                                      │
       │                        Information Extractor (Gemini) ──► Collect page ───────┤
       │                                        ▲                                      │
       │                        Google Gemini Chat Model                              │
       │                                                                               │
       └──► Google Sheets Read ──► Mark existing rows ────────────────────────────────┤
                                                                                       │
       Emit collected ──► Code (flatten rows) ──► Mark new rows ──► Merge ◄────────────┘
                                                                         │
                                                                         ▼
                                              Filter duplicates ──► Google Sheets Append
```

## Files

| File | Description |
|------|-------------|
| `Ikman cars.json` | n8n workflow (no API keys or credentials; uses placeholders). |
| `README.md` | This file. |

## Security

- **No keys or credentials** are stored in the repo. The workflow uses placeholder credential references and `YOUR_GOOGLE_SHEET_ID`; you must add your own Gemini API key, Google Sheets OAuth2, and spreadsheet ID in n8n.
- Keep your n8n instance and credentials secure; only share the spreadsheet with accounts that should have access.

## License

Use and modify as you like. Respect ikman.lk’s terms of use and robots.txt when running the scraper.
