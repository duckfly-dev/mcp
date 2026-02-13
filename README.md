<p align="center">
  <img src="https://unpkg.com/@duckfly/mcp/icons/duck.png" alt="Duckfly" width="140" />
</p>

<h1 align="center">Duckfly MCP</h1>

<p align="center">
  Run a local MCP server from your Duckfly API spec. Tools, prompts, and resources are generated from your documentation; tool calls are forwarded to your API.
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@duckfly/mcp"><img src="https://img.shields.io/npm/v/@duckfly/mcp.svg" alt="npm version" /></a>
  <a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT" /></a>
</p>

## Overview

**Duckfly MCP** is a local MCP (Model Context Protocol) server that uses the spec produced by Duckfly Core for your documented API. It exposes **tools**, **prompts**, and **resources** to AI clients and forwards tool calls to your backend.

Use the same Duckfly token as the proxy plugin. The spec is fetched once at startup from Duckfly Core.

## Why use Duckfly MCP

- Expose your API as MCP tools so AI assistants can call your endpoints
- No manual MCP spec: tools and schemas come from your Duckfly documentation
- Optional auth: None, Basic, Bearer, or OAuth (JWT via JWKS)
- JSON-RPC over HTTP with optional SSE (single global stream or streamable per-request responses)

## Quick Start

### Global install

```bash
npm install -g @duckfly/mcp
```

### Or run with npx

```bash
npx @duckfly/mcp
```

The CLI will ask for:

| Prompt | Description | Default |
|---|---|---|
| **Token** | Your Duckfly application token (same as proxy) | — |
| **MCP Port** | Port where the MCP server listens | `9090` |
| **Target URL** | Base URL of your API (tool calls are sent here) | `http://localhost:3000` |
| **Auth mode** | None / Basic / Bearer / OAuth | `none` |

Configuration is saved in `.duckfly-mcp.json`.

## How it works

```
┌─────────────┐           Duckfly Spec         ┌──────────────┐
│ Duckfly MCP │ <───────────────────────────── │ Duckfly Core │
│ (local)     │                                │ (your spec)  │
└──────┬──────┘                                └──────────────┘
       │
       │ Tool calls (e.g. POST /api/sample/123)
       ▼
┌──────────────┐
│ Your API     │  (Target URL)
└──────────────┘
```

1. At startup, the plugin fetches the MCP spec from Duckfly Core (tools, prompts, resources).
2. AI clients connect to your local MCP server and see those tools.
3. When a client invokes a tool, the server calls **Target URL + path** (e.g. `http://localhost:3001` + `/api/sample/123`).
4. The response is returned to the client; the body includes a `requested` field (method + URL) for debugging.

**If you get 404 on a tool call:** the target server does not have that route. Ensure the service at Target URL is the same API documented in Duckfly, and if your API uses a prefix (e.g. `/v1`), set Target URL to `http://localhost:3001/v1`.

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/mcp` | Open the global SSE channel (one at a time). |
| POST | `/mcp` | JSON-RPC. With `Accept: text/event-stream` you get a streamed response; otherwise, if the global SSE is open, the response is sent there and the POST gets `202`; else you get `application/json`. |
| DELETE | `/mcp` | End the session (requires `Mcp-Session-Id`). |
| GET | `/.well-known/oauth-protected-resource/mcp` | OAuth protected-resource metadata (only when OAuth is enabled). |
| GET | `/health` | Health check (`{ "ok": true }`). |

## License

MIT License

This license applies only to the Duckfly MCP package. The Duckfly platform and services are governed by separate terms.

<p align="center">Made with 🦆 by Duckfly</p>
