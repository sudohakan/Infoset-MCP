<div align="center">

# infoset-mcp

**Model Context Protocol server that gives AI assistants structured access to Infoset CRM tickets, contacts and companies.**

[![CI](https://github.com/sudohakan/infoset-mcp/actions/workflows/ci.yml/badge.svg)](https://github.com/sudohakan/infoset-mcp/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/sudohakan/infoset-mcp?style=flat-square)](https://github.com/sudohakan/infoset-mcp/releases)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen?style=flat-square)](https://nodejs.org)

</div>

## Features

- Wraps the [Infoset](https://infoset.app) CRM API into 18 MCP tools: tickets, contacts, companies, SLA, email threads and batch operations.
- **API key auth**: stateless `X-API-Key` header, no token refresh, no login flow.
- **Rate limit aware**: monitors `x-ratelimit-remaining`, pauses 60s when remaining drops below 2.
- **Retry with backoff**: a 429 response triggers up to 3 retries, 60s apart.
- **Batch tools**: fetch multiple tickets, logs, contacts or companies in one call, 5 requests in parallel, with deduplication for contacts and companies.
- 3 dependencies: `@modelcontextprotocol/sdk`, `axios`, `zod`.

## Installation

```bash
git clone https://github.com/sudohakan/infoset-mcp.git
cd infoset-mcp
npm install
cp .env.example .env     # set INFOSET_API_KEY
```

Register it in your MCP client. The env-wrapper reads credentials from `.env`, so no secrets live in the client config:

```json
{
  "mcpServers": {
    "infoset": {
      "command": "/absolute/path/to/infoset-mcp/scripts/env-wrapper.sh",
      "args": ["/absolute/path/to/infoset-mcp/src/mcp-server.mjs"],
      "env": {}
    }
  }
}
```

### Configuration

| Variable | Required | Default | Description |
|---|:---:|---|---|
| `INFOSET_API_KEY` | Yes | none | Infoset API key (sent as `X-API-Key` header) |
| `INFOSET_BASE_URL` | No | `https://api.infoset.app` | API base URL |
| `INFOSET_USER_ID` | No | none | Your Infoset user ID, the default owner for `infoset_list_tickets` and `infoset_get_ticket_stats`. API-key auth cannot resolve the caller (`/v1/users/me` answers 401), so without it those tools return every owner's tickets |

## Usage

### Ticket operations

| Tool | Description |
|---|---|
| `infoset_list_tickets` | List tickets with status, owner, date, sort filters and pagination |
| `infoset_get_ticket` | Get full ticket detail by ID |
| `infoset_search_tickets` | Search tickets by keyword with status and priority filters |
| `infoset_create_ticket` | Create a ticket with subject, contact, priority, content |
| `infoset_update_ticket` | Update status, priority, owners, subject, stage, pipeline |
| `infoset_get_ticket_logs` | Get activity logs for a ticket |
| `infoset_add_ticket_note` | Add a note to the timeline, private by default, HTML allowed |
| `infoset_delete_ticket_log` | Delete a log entry, such as a note added by mistake |
| `infoset_get_ticket_stats` | Ticket count breakdown by status |

### Contacts, companies, email and SLA

| Tool | Description |
|---|---|
| `infoset_get_contact` | Get contact details by ID |
| `infoset_list_contacts` | List or search contacts by name, email, phone |
| `infoset_get_company` | Get company details by ID |
| `infoset_get_email` | Get email thread content by email ID |
| `infoset_get_sla_breaches` | Get SLA breach data for a ticket |

### Batch operations

| Tool | Description |
|---|---|
| `infoset_batch_get_tickets` | Get multiple tickets in parallel (concurrency 5) |
| `infoset_batch_get_ticket_logs` | Get logs for multiple tickets in parallel |
| `infoset_batch_get_contacts` | Get multiple contacts with auto-deduplication |
| `infoset_batch_get_companies` | Get multiple companies with auto-deduplication |

### Reference codes

| | 1 | 2 | 3 | 4 |
|---|---|---|---|---|
| **Status** | Open | Pending | Resolved | Closed |
| **Priority** | Low | Normal | High | Urgent |

### Development

```bash
npm test               # test suite
npm run test:coverage  # with coverage report
npm start              # start the server standalone
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for code style and pull request guidelines, and [SECURITY.md](SECURITY.md) for credential handling and vulnerability reporting.

## Project structure

```
src/mcp-server.mjs     single-file server: tools, API client, rate limit monitor, retry handler
scripts/env-wrapper.sh  loads .env, then starts the server
tests/                 Jest suite (mcp-server.test.mjs)
```

```
MCP client (Claude, Cursor)  <--stdio-->  infoset-mcp (Node.js)  <--HTTPS-->  Infoset API
```

Transport is stdio (spawned by the MCP client, JSON over stdin/stdout, logs to stderr). Every request carries the `X-API-Key` header.

## License

MIT, see [LICENSE](LICENSE).
