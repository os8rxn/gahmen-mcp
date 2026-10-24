# Gahmen MCP Server

MCP (Model Context Protocol) server for Singapore's [data.gov.sg](https://data.gov.sg) APIs and the [SingStat Table Builder](https://tablebuilder.singstat.gov.sg) APIs, providing easy access to government datasets, collections and statistical tables.

## Features

- Access to data.gov.sg collections and datasets
- Search within datasets using the CKAN datastore API
- Dataset download functionality with filtering support
- Search, metadata and data retrieval for SingStat Table Builder tables
- Built-in request pacing and automatic retry on HTTP 429

## Available Tools

### data.gov.sg — Collections

- `datagovsg_list_collections` - List all collections on data.gov.sg
- `datagovsg_get_collection` - Get metadata for a specific collection

### data.gov.sg — Datasets

- `datagovsg_list_datasets` - List all datasets on data.gov.sg
- `datagovsg_get_dataset_metadata` - Get metadata for a specific dataset
- `datagovsg_search_dataset` - Search for data within a dataset using the CKAN datastore
- `datagovsg_initiate_download` - Start downloading a dataset with optional filtering
- `datagovsg_poll_download` - Check download status and get the download URL

### SingStat Table Builder

- `singstat_search_resources` - Search for SingStat tables by keyword (works best with single words)
- `singstat_get_metadata` - Get metadata for a specific SingStat table
- `singstat_get_table_data` - Get data from a SingStat table with optional filtering

> Not covered: data.gov.sg's real-time APIs (weather, PSI, PM2.5, UV, flood alerts, taxi/carpark availability, traffic images, NLB). Those are served from `api-open.data.gov.sg/v2/real-time/api/...` and are not exposed by this server yet.

## Installation

```bash
npm install
```

## Run locally (stdio)

```bash
npm install
npm run build
npm start          # runs node dist/stdio.js
```

To use it from Claude Code:

```bash
claude mcp add --scope user gahmen -- node /absolute/path/to/gahmen-mcp/dist/stdio.js
```

Claude Desktop: add the same `node … dist/stdio.js` command under `mcpServers` in its config.

## Development

```bash
npx @smithery/cli dev
```

## Build

```bash
npx @smithery/cli build
```

## Usage Examples

### Search a Dataset

```javascript
// Search HDB resale flat prices (dataset ID starts with d_)
datagovsg_search_dataset({
  resource_id: "d_8b84c4ee58e3cfc0ece0d773c8ca6abc",
  filters: { town: "ANG MO KIO" },
  sort: "month desc",
  limit: 10
})
```

### Get Collection with Datasets

```javascript
// Get collection 189 with all dataset metadata
datagovsg_get_collection({
  collectionId: "189",
  withDatasetMetadata: true
})
```

### Get SingStat Table Data

```javascript
// Indicators On Population, Annual
singstat_get_table_data({
  resourceId: "M810001",
  timeFilter: "2023,2024"
})
```

## Upstream API Hosts

| Tools | Host |
|---|---|
| Collections / datasets metadata | `https://api-open.data.gov.sg/v2/public/api` |
| CKAN datastore search | `https://api-open.data.gov.sg/api/action` |
| Downloads | `https://api-open.data.gov.sg/v1/public/api` |
| SingStat | `https://tablebuilder.singstat.gov.sg/api/table` |

The older `api-production.data.gov.sg` and `data.gov.sg/api/action` hosts now sit behind a browser challenge and return HTTP 403 to API clients, so this server uses `api-open.data.gov.sg` for everything.

## API Rate Limits

data.gov.sg rejects bursts with `429 TOO_MANY_REQUESTS` and asks clients to "try again in 10 seconds". The exact quota is not published in a form this project could verify, so the server uses a conservative default budget rather than a documented limit:

- Default budget: 5 requests per 10-second window, one budget each for data.gov.sg and SingStat
- A tool call over budget waits for the next slot (at most 10 s) instead of calling the API
- If the API still answers 429, the request is retried once after the delay it asks for (`Retry-After` header or the "try again in N seconds" message, default 10 s, capped at 30 s)

data.gov.sg offers API keys for higher limits, but this server does not send one.

## No Authentication Required

data.gov.sg and SingStat APIs are public and don't require authentication.

## License

MIT
